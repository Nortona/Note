
![[Pasted image 20230918033320.png]]
点播服务普遍采用了HTTP作为流媒体协议，H.264作为视频编码格式，AAC作为音频编码格式。
采用HTTP作为点播协议有以下两点优势：一方面，HTTP是基于TCP协议的应用层协议，媒体传输过程中不会出现丢包等现象，从而保证了视频的质量；
另一方面，HTTP被绝大部分的Web服务器支持，因而流媒体服务机构不必投资购买额外的流媒体服务器，从而节约了开支。
点播服务采用的封装格式有多种：MP4，FLV，F4V等，它们之间的区别不是很大。
视频编码标准和音频编码标准是H.264和AAC。这两种标准分别是当今实际应用中编码效率最高的视频标准和音频标准。
视频播放器方面，无一例外的都使用了Flash播放器。


# 流程
综合来说播放器的基础工作步骤如下：

1.解协议（读取文件）

2.解封装

3.音视频分离

4.音视频分别解码

5.音视频同步

6.输出数据解码后的音视频数据

7.渲染图像和播放音频
![[Pasted image 20230808170453.png]]







# 问题
## 1、音频未播放完毕，就退出线程


```c++
// 文件数据已经读取完毕
if (buffer.len <= 0) {

	// 剩余的样本数量
	int samples = buffer.pullLen / BYTES_PER_SAMPLE;
	
	// SAMPLE_RATE 采样率，SAMPLE_RATE单位时间为秒，为得到好，毫秒*1000
	int ms = samples * 1000 / SAMPLE_RATE;

	SDL_Delay(ms);

	break;
```

### 2、使用代码重采样后文件大小与FFmpeg命令行转换的文件大小不同

输入缓冲区的样本数量为1024，采样率SampleRate为44100,
输出的采样率SampleRate 为 48000，输出缓冲区的样本数量outNbSamples应该为`outNbSamples = av_rescale_rnd(outSampleRate, inNbSamples, inSampleRate, AV_ROUND_UP)`

```
 inSampleRate     inSamples
 ------------- = -----------
 outSampleRate    outSamples
 
 outSamples = outSampleRate * inSamples / inSampleRate
```

### 3、最后缓冲区存在数据未写入

检查一下输出缓冲区是否还有残留的样本（已经重采样过的，转换过的）

in and in_count can be set to 0 to flush the last few samples out at the end.

```c++
while ((ret = swr_convert(ctx,
						  outAudioData, 
						  outNbSamples,
						  nullptr, 
						  0)) > 0)

{
	outPcmFile.write((char *) outAudioData[0], ret * outBytesPerSample);
}
```

### 4、 播放下一个时崩溃
共用了像素数据image
在停止后应该释放
```c++
void VideoWidget::freeImage() {
    if (_image) {
        av_free(_image->bits());
        delete _image;
        _image = nullptr;
    }
}
```

### 5、播放下一个时有之前的残留

主要由于多线程，主线程通知结束了，但是子线程仍然将decode后的视频帧传给渲染模块，进行判断，若为停止状态，则不接受任何渲染
```c++
if (player->getState() == VideoPlayer::Stopped) return;
```

### 6、偶尔视频出现黑屏

是由于初始化时
```c++
// 初始化音频信息
_hasAudio = initAudioInfo() >= 0;
// 初始化视频信息
_hasVideo = initVideoInfo() >= 0;
if (!_hasAudio && !_hasVideo) {
	fataError();
	return;
}
// 到此为止，初始化完毕
emit initFinished(this);
// 改变状态
setState(Playing);
```

### 7、时间问题
ffmpeg的时间由
- **时间戳**，类型是int64_t
- **时间基**（time base\unit），是时间戳的单位，类型是AVRational
组成，两者相乘为现实时间，时间基为AVRational结构体，包括
- **分子num**
- **分母den**

FFmpeg时间 与 现实时间的转换

1> 现实时间 = 时间戳 * (时间基的分子 / 时间基的分母)

2> 现实时间 = 时间戳 * av_q2d(时间基)
```c++
int VideoPlayer::getDuration() {
    return _fmtCtx ? round(_fmtCtx->duration * av_q2d(AV_TIME_BASE_Q)) : 0;
    // return _fmtCtx ? round(_fmtCtx->duration / 1000000.0) : 0;

}
```

3> 时间戳 = 现实时间 / (时间基的分子 / 时间基的分母)

4> 时间戳 = 现实时间 / av_q2d(时间基)

```c++
static inline double av_q2d(AVRational a){
    return a.num / (double) a.den;
}
```


### 8、解码过快
解码后的数据存放在内存中，如果视频太大，会对内存造成较大压力
```c++
#define AUDIO_MAX_PKT_SIZE 1000
#define VIDEO_MAX_PKT_SIZE 500

int vSize = _vPktList.size();
int aSize = _aPktList.size();

if (vSize >= VIDEO_MAX_PKT_SIZE ||
	aSize >= AUDIO_MAX_PKT_SIZE) {
	continue;
}
```

### 9、音视频同步

（有三个同步解决方案： 音频为基准， 视频为基准，时钟为基准）
本次采用的将音频为基准，原因音频不容易跟视频。

由于音频解码速度由SDL控制
因此采用视频同步到音频

```c++
typedef struct AVPacket {
    /**
     * A reference to the reference-counted buffer where the packet data is
     * stored.

     * May be NULL, then the packet data is not reference-counted.

     */
    AVBufferRef *buf;
    /**

     * Presentation timestamp in AVStream->time_base units; the time at which

     * the decompressed packet will be presented to the user.

     * Can be AV_NOPTS_VALUE if it is not stored in the file.

     * pts MUST be larger or equal to dts as presentation cannot happen before

     * decompression, unless one wants to view hex dumps. Some formats misuse

     * the terms dts and pts/cts to mean something different. Such timestamps

     * must be converted to true pts/dts before they are stored in AVPacket.
     */
    int64_t pts;
```

利用AVPacket中的pts

```c++
// 保存音频时钟

if (pkt.pts != AV_NOPTS_VALUE) {

	_aTime = av_q2d(_aStream->time_base) * pkt.pts;

	// 通知外界：播放时间点发生了改变

	emit timeChanged(this);

}


// 视频时钟
if (pkt.dts != AV_NOPTS_VALUE) {

	_vTime = av_q2d(_vStream->time_base) * pkt.dts;
}
```

视频渲染之前，进行判断
如果视频包过早被解码出来，就需要等待对应的音频时钟到达再进行渲染
```c++
while (_vTime > _aTime && _state == Playing)
{

//   SDL_Delay(5);

}
```


### 10、跳转seek操作

av_seek_frame

```c++
// 处理seek操作

if (_seekTime >= 0) {
	int streamIdx;
	if (_hasAudio) { // 优先使用音频流索引
		streamIdx = _aStream->index;
	} else {
		streamIdx = _vStream->index;
	}
	// 现实时间 -> 时间戳
	AVRational timeBase = _fmtCtx->streams[streamIdx]->time_base;
	int64_t ts = _seekTime / av_q2d(timeBase);
	ret = av_seek_frame(_fmtCtx, streamIdx, ts, AVSEEK_FLAG_BACKWARD);

	if (ret < 0) { // seek失败
		qDebug() << "seek失败" << _seekTime << ts << streamIdx;
		_seekTime = -1;
	} else {
		qDebug() << "seek成功" << _seekTime << ts << streamIdx;
		_vSeekTime = _seekTime;
		_aSeekTime = _seekTime;
		_seekTime = -1;
		
		// 恢复时钟, 避免while循环中循环等待，赋值二者相等退出循环
		_aTime = 0;
		_vTime = 0;
		
		// 清空之前读取的数据包
		clearAudioPktList();
		clearVideoPktList();
	}
```

### 11、花屏现象
除了音频队列，解码器中也保留了上一帧视频的信息，而视频发生跳转时，解码器中保留的信息会对解码当前帧产生影响

例如：从3s处跳转到15min处，则3s处的包还残留在队列和解码器中，与15min处的结合会产生"花屏"

因此，清空队列的时候我们也要同时清空解码器的数据

这里的操作是，往队列中放入一个特殊的 packet，当解码线程取到这个 packet 的时候，就执行清除解码器数据的操作

在解码器类头文件中定义“清空”队列包的宏

#define FLUSH_DATA "FLUSH"



### 12、seek后前面几帧速度快

可能由于音频帧出现问题



![[Pasted image 20230918032313.png]]



# 调试输出
## 1、提取各个流的信息结果
avformat_oopen_input->avformat_find_stram_info

```
_fmtCtx 0x5bee1f0

Input #0, mov,mp4,m4a,3gp,3g2,mj2, from 'C:/Users/12543/Videos/walking-dead.mp4':

Metadata:
major_brand : isom
minor_version : 512
compatible_brands: isomiso2avc1mp41
encoder : Lavf58.45.100
Duration: 00:00:10.02, start: 0.000000, bitrate: 602 kb/s

Stream #0:0(und): Video: h264 (High) (avc1 / 0x31637661), yuv420p, 720x404 [SAR 1:1 DAR 180:101], 467 kb/s, 24 fps, 24 tbr, 12288 tbn, 48 tbc (default)
Metadata:
handler_name : VideoHandler

Stream #0:1(eng): Audio: aac (LC) (mp4a / 0x6134706D), 48000 Hz, stereo, fltp, 128 kb/s (default)
Metadata:
handler_name : SoundHandle
```

