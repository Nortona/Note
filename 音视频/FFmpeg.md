## **比特率**
显示一段视频每秒所需的比特数量就是它的**比特率**。
> 比特率 = 宽 * 高 * 比特深度 * 帧每秒
> 
# 安装

1. 下载源码
```bash
git clone git://source.ffmpeg.org/ffmpeg.git ffmpeg
```
2. 创建配置文件[[音视频基础知识]]
```bash
./configure   --enable-shared  --prefix=/usr/local/ffmpeg  --enable-gpl --enable-libx264  --enable-libx265   

```
3. 支持库安装
```bash
sudo apt-get install -y autoconf automake build-essential git libass-dev libfreetype6-dev libsdl2-dev libtheora-dev libtool libva-dev libvdpau-dev libvorbis-dev libxcb1-dev libxcb-shm0-dev libxcb-xfixes0-dev pkg-config texinfo wget zlib1g-dev

sudo apt install libavformat-dev
sudo apt install libavcodec-dev
sudo apt install libswresample-dev
sudo apt install libswscale-dev
sudo apt install libavutil-dev
sudo apt install libsdl1.2-dev
```

4. make && make install
5. 添加环境变量
```bash
sudo vi /etc/ld.so.conf
# 在其中添加路径：/usr/local/ffmpeg/lib

sudo ldconfig#更新环境变量
```
# 常用工具
ffmpeg：多媒体编解码
ffprobe：内容分析
ffplay：播放器

# 小试
## 1. ffmpeg视频文件再编码
### 流程
读取文件-解封装-解码-转换参数-新编码-封装-写入文件

### 视频编解码 vs 容器
可以将容器视为包含视频（也很可能包含音频）元数据的包装格式，压缩过的视频可以看成是它承载的内容。
通常，视频文件的格式定义其视频容器。例如，文件 video.mp4 可能是 MPEG-4 Part 14 容器，一个叫 video.mkv 的文件可能是 matroska。我们可以使用 ffmpeg 或 mediainfo 来完全确定编解码器和容器格式。


 - 通过re-encoding media streams对视频文件再编码:
```shell
ffmpeg -i input.avi output.mp4
```
-   Set the video bitrate of the output file to 64 kbit/s:
- b ：音频与视频码率 默认200kbit/s 
	- b:v 视频码率
	- b:a 音频码率
```shell
ffmpeg -i input.avi -b:v 64k -bufsize 64k output.mp4   
```
-   Force the frame rate of the output file to 24 fps:
```shell
ffmpeg -i input.avi -r 24 output.mp4
```
-   Force the frame rate of the input file (valid for raw formats only) to 1 fps and the frame rate of the output file to 24 fps:  
```shell
ffmpeg -r 1 -i input.m2v -r 24 output.mp4
```

## 2. ffprobe

### 常用命令
#### 输出数据包信息
```shell
ffprobe -show_packets -show_data mv.mp4
# show_data 显示具体数据
```

#### 封装格式
```shell
ffprobe -show_format mv.mp4
```

#### 输出帧信息
```shell
ffprobe -show_frames mv.mp4
```

#### 输出流信息
```shell
ffprobe -show_streams mv.mp4
```

#### -of 控制输出格式
```shell
ffprobe -of xml（csv/json/ini/flat） -show_streams mv.mp4
```

#### -select_streams v/a/s  筛选流
```shell
ffprobe -show_farmes -select_streams v mv.mp4
```

## ffplay
![[3285193-891f9666dcab2f1b.webp]]
![[3285193-d8e1675728065e92.webp]]

### 显示模式
可用的模式值： 0 显示视频，  1 显示音频波形， 2 显示音频频谱。缺省为 0 ，如果视频不存在则自动选择 2
```shell
 ffplay -showmode 1 mv.mp4
```


# 通用视频编解码器
## 第一步 - 图片分区
第一步是**将帧**分成几个**分区**，**子分区**及其以外。
![[Pasted image 20230505144112.png]]

**但是为什么呢？有许多原因，比如，当我们分割图片时，我们可以更精确的处理预测，在微小移动的部分使用较小的分区，而在静态背景上使用较大的分区。

通常，编解码器**将这些分区组织**成切片（或瓦片），宏（或编码树单元）和许多子分区。这些分区的最大大小有所不同，HEVC 设置成 64x64，而 AVC 使用 16x16，但子分区可以达到 4x4 的大小。

还记得我们学过的**帧的分类**吗？！好的，你也可以**把这些概念应用到块**，因此我们可以有 I 切片，B 切片，I 宏块等等。


## 第二步 - 预测

一旦我们有了分区，我们就可以在它们之上做出预测。对于帧间预测，我们需要**发送运动向量和残差**；至于帧内预测，我们需要**发送预测方向和残差**。


## 第三步 - 转换

在我们得到残差块（`预测分区-真实分区`）之后，我们可以用一种方式**变换**它，这样我们就知道**哪些像素我们应该丢弃**，还依然能保持**整体质量**。这个确切的行为有几种变换方式。
1. 离散余弦变换（DCT）
主要功能有：
- 将**像素**块**转换**为相同大小的**频率系数块**。
- **压缩**能量，更容易消除空间冗余。
- **可逆的**，也意味着你可以逆转为像素。

来看下面的**像素块**（8x8）
![[Pasted image 20230505151527.png]]
下面是其渲染的块图像（8x8）：
![[Pasted image 20230505151538.png]]
对这个像素块**应用 DCT** 时， 得到如下**系数块**（8x8）：
![[Pasted image 20230505151549.png]]
渲染这个系数块，就会得到这张图片：
![[Pasted image 20230505151609.png]]
第一个系数被称为表示输入数组中**所有样本**的 DC 系数，**类似于平均值**。
这个系数块有一个有趣的属性，显示为高频率组件和低频率分离。![[Pasted image 20230505151715.jpg]]

在一张图像中，**大多数能量**会集中在[低频率](https://www.iem.thm.de/telekom-labor/zinke/mk/mpeg2beg/whatisit.htm)，所以如果我们将图像变换成频率组件，并**丢掉高频率系数**，我们就能**减少需要描述图像的数据量**，而不会牺牲太多的图像质量。
> 频率是指信号变化的速度。

使用 DCT 将原始图像转换为其频率（系数块），然后丢掉最不重要的系数。

首先，我们将它转换为其**频域**。
![[Pasted image 20230505160415.png]]

然后我们丢弃部分（67%）系数，主要是它的右下角部分。
![[Pasted image 20230505160440.png]]
然后我们从丢弃的系数块重构图像（记住，这需要可逆），并与原始图像相比较。
![[Pasted image 20230505160451.png]]


# FFmpeg转封装

## MP4容器

- MP4文件由许多个Box与FullBox组成

- 每个Box由Header和Data两部分组成

- FullBox是Box的扩展，其在Box结构的基础上，在Header中增加8位version标志和24位的flags标志

- Header包含了整个Box的长度的大小（size）和类型（type），当size等于0时，代表这个Box是文件的最后一个Box。当size等于1时，说明Box长度需要更多的位来描述，在后面会定义一个64位的largesize用来描述Box的长度。当Type为uuid时，说明这个Box中的数据是用户自定义扩展类型

- Data为Box的实际数据，可以是纯数据，也可以是更多的子Box

- 当一个Box中Data是一系列的子Box时，这个Box又可以称为Container（容器）Box


## FLV
在网络的直播与点播场景中，FLV也是一种常见的格式，FLV是Adobe发布的一种可以作为直播也可以作为点播的封装格式，其封装格式非常简单，均以FLVTAG的形式存在，并且每一个TAG都是独立存在的，接下来就来详细介绍一下FLV标准。
若原视频的编码受到flv容器支持，则可以直接转封装
```bash
ffmpeg -i input_ac3.mp4 -c copy -f flv output.flv
```
若不支持，则需要进行转码
```shell
./ffmpeg -i input_ac3.mp4 -vcodec copy -acodec aac -f flv output.flv
```

### FFmpeg生成带关键索引的FLV
在网络视频点播文件为FLV格式文件时，人们常用yamdi工具先对FLV文件进行一次转换，主要是将FLV文件中的关键帧建立一个索引，并将索引写入Metadata头中，这个步骤用FFmpeg同样也可以实现，使用参数add_keyframe_index即可：

```bash
ffmpeg -i input.mp4 -c copy -f flv -flvflags add_keyframe_index output.flv
```

## M3U8
M3U8是一种常见的流媒体格式，主要以文件列表的形式存在，既支持直播又支持点播，尤其在Android、iOS等平台最为常用

## HLS
FFmpeg中自带HLS的封装参数，使用HLS格式即可进行HLS的封装，但是生成HLS的时候有各种参数可以进行参考，例如设置HLS列表中切片的前置路径、生成HLS的TS切片时设置TS的分片参数、生成HLS时设置M3U8列表中保存的TS个数等
![[Image00074.jpg]]

常规的从文件转换HLS直播时，使用的参数如下：
```SHELL
./ffmpeg -re -i input.mp4 -c copy -f hls -bsf:v h264_mp4toannexb output.m3u8
```
因为默认是HLS直播，所以生成的M3U8文件内容会随着切片的产生而更新，如果仔细观察，会发现命令行中多了一个参数==“-bsf：v h264_mp4toannexb”==，这个参数的作用是==将MP4中的H.264数据转换为H.264AnnexB标准的编码==，AnnexB标准的编码常见于实时传输流中。如果源文件为FLV、TS等可作为直播传输流的视频，则不需要这个参数。
![[Pasted image 20230505223756.png]]

## 视频文件切片
视频文件切片与HLS基本类似，但是HLS切片在标准中只支持TS格式的切片，并且是直播与点播切片，既可以使用segment方式进行切片，也可以使用ss加上t参数进行切片
![[Image00075.jpg]]
![[Image00076.jpg]]

### segment_format指定切片文件的格式
通过使用segment_format来指定切片文件的格式，前面讲述过HLS切片的格式主要为MPEGTS文件格式，那么在segment中，可以根据segment_format来指定切片文件的格式，其既可以为MPEGTS切片，也可以为MP4切片，还也可以为FLV切片等，
```shell
ffmpeg -re -i input.mp4 -c copy -f segment -segment_format mp4 test_output-%d.mp4
```

### FFmpeg使用ss与t参数进行切片
使用ss指定剪切开头部分
```shell
ffmpeg -ss 10 -i input.mp4 -c copy output.ts
```
### 使用t指定视频总长度
```shell
ffmpeg -i input.mp4 -c copy -t 10 -copyts output.mp4
```

## 音视频流抽取
### 抽取音频流
```shell
ffmpeg -i input.mp4 -vn -acodec copy output.aac
```
### 抽取H.264视频流
```shell
ffmpeg -re -i input.mp4 -c copy -f mpegts output.ts
```
