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


# 通用视频编解码器 （编码步骤）
## 第一步 - 图片分区
第一步是**将帧**分成几个**分区**，**子分区**及其以外。
![[Pasted image 20230505144112.png]]

**但是为什么呢？有许多原因，比如，当我们分割图片时，我们可以更精确的处理预测，在微小移动的部分使用较小的分区，而在静态背景上使用较大的分区。

通常，编解码器**将这些分区组织**成切片（或瓦片），宏（或编码树单元）和许多子分区。这些分区的最大大小有所不同，HEVC 设置成 64x64，而 AVC 使用 16x16，但子分区可以达到 4x4 的大小。

还记得我们学过的**帧的分类**吗？！好的，你也可以**把这些概念应用到块**，因此我们可以有 I 切片，B 切片，I 宏块等等。


## 第二步 - 预测（帧内预测   帧间预测）

一旦我们有了分区，我们就可以在它们之上做出预测。对于帧间预测，我们需要**发送运动向量和残差**；至于帧内预测，我们需要**发送预测方向和残差**。

帧内预测，对一帧视频进行编码，I帧。

帧间预测，几帧视频进行编码，P帧、B帧。这部分有运动估计，优化的难点。


## 第三步 - 转换（DTC   量化编码）

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


## 第四步 - 熵编码
经常出现的短码表示，偶尔出现的长码表示。

一般是cabac方式。

原始值-预测值=差值，对差值编码。


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


# 音视频转码

## H.264编码
编码参数
![[Image00079.jpg]]
![[Image00080.jpg]]

### preset
编码器预设参数设置preset: 
- ultrafast
- superfast
- veryfast
- faster
- fast
- medium
- slow
- veryslow
- placebo


```shell
ffmpeg -i input.mp4 -vcodec libx264 -preset ultrafast -b:v 2000k output.mp4
```
```shell
ffmpeg -i input.mp4 -vcodec libx264 -preset medium -b:v 2000k output.mp4
```


### profile + level
#### profile:
- baseline
- extented
- main
- high
- high10
- high422
- high444

high10与high的区别主要在于支持9位10位采样深度
high444与high422区别在于支持4:4:4色度格式
![[Image00082.jpg]]


#### level
level设置则与标准的ISO-14496-Part10参考中的Annex A中描述的表格完全相同
![[Image00083.jpg]]
![[Image00084.jpg]]

使用baseline profile编码的H.264视频不会包含B Slice，而使用main profile、high profile编码出来的视频，均可以包含B Slice

使用FFmpeg编码生成baseline与high两种profile的视频：
```shell
ffmpeg -i input.mp4 -vcodec libx264 -profile:v baseline -level 3.1 -s 352x288 -an -y -t 10 output_baseline.ts 

ffmpeg -i input.mp4 -vcodec libx264 -profile:v high -level 3.1 -s 352x288 -an -y -t 10 output_high.ts

```

使用ffprobe查看这两个文件中包含B帧的情况：
```shell
ffprobe -v quiet -show_frames -select_streams v output_baseline.ts |grep "pict_type=B"|wc -l 0 

ffprobe -v quiet -show_frames -select_streams v output_high.ts |grep "pict_type=B"|wc -l 140
```

![[Pasted image 20230506212104.png]]

当进行实时流媒体直播时，采用baseline编码相对main或high的profile会更可靠些；
但适当地加入B帧能够有效地降低码率，所以应根据需要与具体的业务场景进行选择。

### 控制场景切换关键帧插入参数sc_threshold
GOP group of pictures

GOP 指的就是两个I帧之间的间隔.

FFmpeg中，通过命令行的-g参数设置以帧数间隔为GOP的长度，但是当遇到场景切换时，例如从一个画面突然变成另外一个画面时，会强行插入一个关键帧，这时GOP的间隔将会重新开始，这样的场景切换在点播视频文件中会频频遇到，如果将点播文件进行M3U8切片，或者将点播文件进行串流虚拟直播时，GOP的间隔也会出现相同的情况，为了避免这种情况的产生，可以通过使用sc_threshold参数进行设定以决定是否在场景切换时插入关键帧。

```shell
ffmpeg -i input.mp4 -c:v libx264 -g 50 -t 60 output.mp4

ffmpeg -i input.mp4 -c:v libx264 -g 50 -sc_threshold 0 -t 60 -y output.mp4
```


### 设置x264内部参数x264opts

控制I帧、P帧、B帧的顺序及出现频率，首先分析一下设置的GOP参数，如果视频GOP设置为50帧，那么如果在这50帧中不希望出现B帧，则客户只需要将x264参数bframes设置为0即可
```shell
ffmpeg -i input.mp4 -c:v libx264 -x264opts "bframes=0" -g 50 -sc_threshold 0 output.mp4
```


如果希望控制I帧、P帧、B帧的频率与规律，可以通过控制GOP中B帧的帧数来实现，P帧的频率可以通过x264的参数b-adapt进行设置。
设置GOP中，每2个P帧之间存放3个B帧：
```shell
ffmpeg -i input.mp4 -c:v libx264 -x264opts "bframes=3:b-adapt=0" -g 50 -sc_threshold 0 output.mp4
```

视频中的==B帧越多，同等码率时的清晰度将会越高，但是B帧越多，编码与解码所带来的复杂度也就越高==，所以合理地使用B帧非常重要，尤其是在进行清晰度与码率衡量时。

### CBR恒定码率设置参数nal-hrd
编码能够设置VBR、CBR的编码模式，VBR为可变码率，CBR为恒定码率。尽管现在互联网上所看到的视频中VBR居多，但CBR依然存在

FFmpeg是通过参数-b：v来指定视频的编码码率的，但是设定的码率是平均码率，并不能够很好地控制最大码率及最小码率的波动，如果需要控制最大码率和最小码率以控制码率的波动，则需要使用FFmpeg的三个参数-b：v、maxrate、minrate。
```shell
ffmpeg -i input.mp4 -c:v libx264 -x264opts "bframes=10:b-adapt=0"  -b:v 1000k -maxrate 1000k -minrate 1000k -bufsize 50k -nal-hrd cbr -g 50 -sc_threshold 0 output.ts
```

- 设置B帧的个数，并且是每两个P帧之间包含10个B帧

- 设置视频码率为1000kbit/s

- 设置最大码率为1000kbit/s

- 设置最小码率为1000kbit/s

- 设置编码的buffer大小为50KB

- 设置H.264的编码HRD信号形式为CBR

- 设置每50帧一个GOP

- 设置场景切换不强行插入关键帧


## FFmpeg硬编解码
当使用FFmpeg进行软编码时，常见的基于CPU进行H.264或H.265编码其相对成本会比较高，CPU编码时的性能也很低，所以出于编码效率及成本考虑，很多时候都会考虑采用硬编码，常见的硬编码包含Nvidia GPU与Intel QSV两种

### Nvidia GPU硬编解码
#### 参数
![[Image00091.jpg]]
![[Image00092.jpg]]

使用Nvidia GPU进行硬编解码
```shell
ffmpeg -hwaccel nvdec -hwaccel_device 0 -hwaccel_output_format cuda  -vcodec h264_cuvid -i mv.mp4 -vf scale_cuda=1920:1080 -vcodec h264_nvenc -acodec copy -f mp4 -y output_nvidia.mp4
```

```
Input #0, mov,mp4,m4a,3gp,3g2,mj2, from 'mv.mp4':
  Metadata:
    major_brand     : isom
    minor_version   : 512
    compatible_brands: isomiso2avc1mp41
    encoder         : Lavf58.28.101
  Duration: 00:03:51.34, start: 0.000000, bitrate: 1635 kb/s
  Stream #0:0(und): Video: h264 (High) (avc1 / 0x31637661), yuv420p(tv, bt709), 1920x1080 [SAR 1:1 DAR 16:9], 1502 kb/s, 24 fps, 24 tbr, 12288 tbn, 48 tbc (default)
    Metadata:
      handler_name    : ISO Media file produced by Google Inc.
      vendor_id       : [0][0][0][0]
  Stream #0:1(eng): Audio: aac (LC) (mp4a / 0x6134706D), 44100 Hz, stereo, fltp, 128 kb/s (default)
    Metadata:
      handler_name    : ISO Media file produced by Google Inc.
      vendor_id       : [0][0][0][0]
Stream mapping:
  Stream #0:0 -> #0:0 (h264 (h264_cuvid) -> h264 (h264_nvenc))
  Stream #0:1 -> #0:1 (copy)
Press [q] to stop, [?] for help
Output #0, mp4, to 'output_nvidia.mp4':
  Metadata:
    major_brand     : isom
    minor_version   : 512
    compatible_brands: isomiso2avc1mp41
    encoder         : Lavf58.76.100
  Stream #0:0(und): Video: h264 (Main) (avc1 / 0x31637661), cuda(tv, bt709, progressive), 1920x1080 [SAR 1:1 DAR 16:9], q=2-31, 2000 kb/s, 24 fps, 12288 tbn (default)
    Metadata:
      handler_name    : ISO Media file produced by Google Inc.
      vendor_id       : [0][0][0][0]
      encoder         : Lavc58.134.100 h264_nvenc
    Side data:
      cpb: bitrate max/min/avg: 0/0/2000000 buffer size: 4000000 vbv_delay: N/A
  Stream #0:1(eng): Audio: aac (LC) (mp4a / 0x6134706D), 44100 Hz, stereo, fltp, 128 kb/s (default)
    Metadata:
      handler_name    : ISO Media file produced by Google Inc.
      vendor_id       : [0][0][0][0]
frame= 5551 fps=437 q=27.0 Lsize=   61613kB time=00:03:51.31 bitrate=2182.0kbits/s speed=18.2x    
video:57844kB audio:3615kB subtitle:0kB other streams:0kB global headers:0kB muxing overhead: 0.250493%

```

`Stream #0:0 -> #0:0 (h264 (h264_cuvid) -> h264 (h264_nvenc))`
使用`h264_cuvid`硬解码与`h264_nvenc`硬编码

可以极大提高速度

### Intel QSV硬编码
参数
![[Image00094.jpg]]
![[Image00095.jpg]]

```shell
ffmpeg -i 10M1080P.mp4 -pix_fmt nv12 -vcodec h264_qsv -an -y output.mp4
```


在使用Intel进行高清编码时，使用AVC编码之后观察码率会比较高，但是使用H.265（HEVC）则能更好地降低同样清晰度的码率，
```shell
ffmpeg -hide_banner -y -hwaccel qsv -i 10M1080P.mp4 -an -c:v hevc_qsv -load_plugin hevc_hw -b:v 5M -maxrate 5M out.mp4
```

### MP3的编码质量设置
在FFmpeg中进行MP3编码采用的是第三方库libmp3lame，所以进行MP3编码时，需要设置编码参数acodec为libmp3lame，
```shell
ffmpeg –i INPUT –acodec libmp3lame OUTPUT.mp3
```
#### MP3基本信息
![[Image00101.jpg]]

```shell
ffmpeg -i input.mp3 -acodec libmp3lame -q:a 8 output.mp3
```
执行完上面这条命令行之后，将生成的output.mp3的码率区间设置在70kbit/s至105kbit/s之间

### AAC
在音视频流中，无论直播与点播，AAC都是目前最常用的一种音频编码格式，例如RTMP直播、HLS直播、RTSP直播、FLV直播、FLV点播、MP4点播等文件中都是常见的AAC音视频。
与MP3相比，AAC是一种编码效率更高、编码音质更好的音频编码格式，常见的使用AAC编码后的文件存储格式为m4a

FFmpeg可以支持AAC的三种编码器具体如下。
- aac：FFmpeg本身的AAC编码实现

- libfaac：第三方的AAC编码器

- libfdk_aac：第三方的AAC编码器

#### acc
```shell
ffmpeg -i input.mp4 -c:a aac -b:a 160k output.aac
```

#### FDK AAC第三方的AAC编解码Codec库
##### 恒定码率（CBR）模式
如果使用libfdk_aac设定一个恒定的码率，改变编码后的大小，并且可以兼容HE-AAC Profile，则可以根据音频设置的经验设置码率，例如如果一个声道使用64kbit/s，那么双声道为128kbit/s，环绕立体声为384kbit/s，这种通常为5.1环绕立体声。可以通过b：a参数进行设置
```shell
ffmpeg -i input.wav -c:a libfdk_aac -b:a 128k output.m4a
```
根据这条命令行可以看出，FFmpeg使用libfdk_aac将input.wav转为恒定码率为128kbit/s、编码为AAC的output.m4a音频文件。

```shell
ffmpeg -i input.mp4 -c:v copy -c:a libfdk_aac -b:a 384k output.mp4
```

根据这条命令行可以看出，FFmpeg将input.mp4的视频文件按照原有的编码方式进行输出封装，将音频以libfdk_aac进行编码，音频通道为环绕立体声，码率为384kbit/s，封装格式为output.mp4。

##### 动态码率（VBR）模式

使用VBR可以有更好的音频质量，使用libfdk_aac进行VBR模式的AAC编码时，可以设置5个级别。

第一列为VBR的类型，第二列为每通道编码后的码率，第三列中有三种AAC编码信息，具体如下。
![[Image00102.jpg]]

·LC：Low Complexity AAC，这种编码相对来说体积比较大，质量稍差

·HE：High-Efficiency AAC，这种编码相对来说体积稍小，质量较好

·HEv2：High-Efficiency AAC version2，这种编码相对来说体积小，质量优


LC、HE、HEv2的推荐参数
![[Image00103.jpg]]
![[Image00104.jpg]]
```shell
ffmpeg -i input.wav -c:a libfdk_aac -vbr 3 output.m4a
# FFmpeg会将input.wav的音频转为音频编码为libfdk_aac的output.m4a音频文件。
```


#### AAC音频质量对比

AAC-LC的音频编码可以采用libfaac、libfdk_aac、FFmpeg内置AAC三种，其质量顺序排列如下。

·libfdk_aac音频编码质量最优

·FFmpeg内置AAC编码次于libfdk_aac但优于libfaac

·libfaac在FFmpeg内置AAC编码为实验品时是除了libfdk_aac之外的唯一选择

