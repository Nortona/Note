
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

### 11、seek后前面几帧速度快

可能由于音频帧出现问题