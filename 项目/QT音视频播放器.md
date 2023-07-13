
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

# 流程
