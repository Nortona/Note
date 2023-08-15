![[Pasted image 20230810021212.png]]

# 格式概述

## isom
isom（ISO Base Media file）是在 MPEG-4 Part 12 中定义的一种基础文件格式，MP4、3gp、QT 等常见的封装格式，都是基于这种基础文件格式衍生的。

## BOX
MP4文件由多个box组成，每个box存储不同的信息，且box之间是树状结构，如下图所示。
![[Pasted image 20230809131119.png]]

box类型有很多，下面是3个比较重要的顶层box：
- ftyp：File Type Box，描述文件遵从的MP4规范与版本；
- moov：Movie Box，媒体的metadata信息，有且仅有一个。
- mdat：Media Data Box，存放实际的媒体数据，一般有多个；
![[Pasted image 20230809131935.png]]

1个box由两部分组成：box header、box body。
1. box header：box的元数据，比如box type、box size。
2. box body：box的数据部分，实际存储的内容跟box类型有关，比如mdat中body部分存储的媒体数据。

当box body中嵌套其他box时，这样的box叫做container box。
![[Pasted image 20230809132108.png]]

## Box Header

字段定义如下：

- type：box类型，包括 “预定义类型”、“自定义扩展类型”，占4个字节；
    - 预定义类型：比如ftyp、moov、mdat等预定义好的类型；
    - 自定义扩展类型：如果type`==`uuid，则表示是自定义扩展类型。size（或largesize）随后的16字节，为自定义类型的值（extended_type）
- size：包含box header在内的整个box的大小，单位是字节。当size为0或1时，需要特殊处理：
    - size等于0：box的大小由后续的largesize确定（一般只有装载媒体数据的mdat box会用到largesize）；
    - size等于1：当前box为文件的最后一个box，通常包含在mdat box中；
- largesize：box的大小，占8个字节；
- extended_type：自定义扩展类型，占16个字节；

### Box vs FullBox
在Box的基础上，扩展出了FullBox类型。相比Box，FullBox 多了 version、flags 字段。
- version：当前box的版本，为扩展做准备，占1个字节；
- flags：标志位，占24位，含义由具体的box自己定义；



## ftyp（File Type Box）
ftyp用来指出当前文件遵循的规范
![[Pasted image 20230809142819.png]]


- major_brand：比如常见的 isom、mp41、mp42、avc1、qt等。它表示“最好”基于哪种格式来解析当前的文件。举例，major_brand 是 A，compatible_brands 是 A1，当解码器同时支持 A、A1 规范时，最好使用A规范来解码当前媒体文件，如果不支持A规范，但支持A1规范，那么，可以使用A1规范来解码；
- minor_version：提供 major_brand 的说明信息，比如版本号，不得用来判断媒体文件是否符合某个标准/规范；
- compatible_brands：文件兼容的brand列表。比如 mp41 的兼容 brand 为 isom。通过兼容列表里的 brand 规范，可以将文件 部分（或全部）解码出来；

下面是常见的几种brand，以及对应的文件扩展名、mime type，
![[Pasted image 20230809142236.png]]


## moov（Movie Box）
Movie Box，存储 mp4 的 metadata，一般位于mp4文件的开头。

moov中，最重要的两个box是 mvhd 和 trak：
- mvhd：Movie Header Box，mp4文件的整体信息，比如创建时间、文件时长等；
- trak：Track Box，一个mp4可以包含一个或多个轨道（比如视频轨道、音频轨道），轨道相关的信息就在trak里。trak是container box，至少包含两个box，tkhd、mdia；


### mvhd( Movie Header Box）
MP4文件的整体信息，跟具体的视频流、音频流无关，比如创建时间、文件时长等。
![[aeef32359fa24afbb5763368ab350bf2.gif]]
字段含义如下：
- creation_time：文件创建时间；
- modification_time：文件修改时间；
- timescale：一秒包含的时间单位（整数）。举个例子，如果timescale等于1000，那么，一秒包含1000个时间单位（后面track等的时间，都要用这个来换算，比如track的duration为10,000，那么，track的实际时长为10,000/1000=10s）；
- duration：影片时长（整数），根据文件中的track的信息推导出来，等于时间最长的track的duration；
- rate：推荐的播放速率，32位整数，高16位、低16位分别代表整数部分、小数部分（[16.16]），举例 0x0001 0000 代表1.0，正常播放速度；
- volume：播放音量，16位整数，高8位、低8位分别代表整数部分、小数部分（[8.8]），举例 0x01 00 表示 1.0，即最大音量；
- matrix：视频的转换矩阵，一般可以忽略不计；
- next_track_ID：32位整数，非0，一般可以忽略不计。当要添加一个新的track到这个影片时，可以使用的track id，必须比当前已经使用的track id要大。也就是说，添加新的track时，需要遍历所有track，确认可用的track id；

### tkhd（Track Box）
![[74d7a11cf4854e069e57df73194762eb.gif]]
单个 track 的 metadata，包含如下字段：
- version：tkhd box的版本；
- flags：按位或操作获得，默认值是7（0x000001 | 0x000002 | 0x000004），表示这个track是启用的、用于播放的 且 用于预览的。
    - Track_enabled：值为0x000001，表示这个track是启用的，当值为0x000000，表示这个track没有启用；
    - Track_in_movie：值为0x000002，表示当前track在播放时会用到；
    - Track_in_preview：值为0x000004，表示当前track用于预览模式；
- creation_time：当前track的创建时间；
- modification_time：当前track的最近修改时间；
- track_ID：当前track的唯一标识，不能为0，不能重复；
- duration：当前track的完整时长（需要除以timescale得到具体秒数）；
- layer：视频轨道的叠加顺序，数字越小越靠近观看者，比如1比2靠上，0比1靠上；
- alternate_group：当前track的分组ID，alternate_group值相同的track在同一个分组里面。同个分组里的track，同一时间只能有一个track处于播放状态。当alternate_group为0时，表示当前track没有跟其他track处于同个分组。一个分组里面，也可以只有一个track；
- volume：audio track的音量，介于0.0~1.0之间；
- matrix：视频的变换矩阵；
- width、height：视频的宽高；



### hdlr（Handler Reference Box）

声明当前track的类型，以及对应的处理器（handler）。

handler_type的取值包括：

- vide（0x76 69 64 65），video track；
- soun（0x73 6f 75 6e），audio track；
- hint（0x68 69 6e 74），hint track；提示跟踪用于描述如何通过特定网络传输发送引用媒体跟踪

### stbl（Sample Table Box）
主要存放了媒体参数(pps、sps、vps等)相关信息和用于解析mdat中视音频数据的关键信息

MP4封装格式，有一些sample和chunk，对于视频，一个sample就是一帧。chunk是包含一个或者多个sample。其实有很多 sample的时间戳是一样的，封装在一个chunk，至少可以省点空间吧。当然每帧的数据长度是不一样的。


- stsd(Sample Description Box）：给出视音频的相关参数信息，有高宽、音量、位深度和每个sample多少个frame
- stco (Chunk Offset Box）：chunk在文件中的偏移
- stsc (Sample To Chunk Box）：每个chunk中包含几个sample，Sample to chunk 的映射表。chunk的size可以是不同的，chunk里面的sample的size也可以是不同的。
	- entry_count：有多少个表项（每个表项，包含first_chunk、samples_per_chunk、sample_description_index信息）；
	- first_chunk：当前表项中，对应的第一个chunk的序号；
	- samples_per_chunk：每个chunk包含的sample数；
	- sample_description_index：指向 stsd 中 sample description 的索引值（参考stsd小节）；
- stsz：每个sample的size（单位是字节），每个Sample大小的表，根据 sample_size 字段，可以知道当前track包含了多少个sample（或帧）。
	- sample_size：默认的sample大小（单位是byte），通常为0。如果sample_size不为0，那么，所有的sample都是同样的大小。如果sample_size为0，那么，sample的大小可能不一样。
	- sample_count：当前track里面的sample数目。如果 sample_size`==`0，那么，sample_count 等于下面entry的条目；
	- entry_size：单个sample的大小（如果sample_size`==`0的话）；
- stts (Decoding Time to Sample Box）：每个sample的时长，时间戳和Sample映射表，主要用来推导每个帧的时长。
	- entry_count：stts 中包含的entry条目数；
	- sample_count：单个entry中，具有相同时长（duration 或 sample_delta）的连续sample的个数。
	- sample_delta：sample的时长（以timescale为计量）
- stss：哪些sample是关键帧，关键帧索引表

## fMP4（Fragmented mp4）
fMP4 跟普通 mp4 基本文件结构是一样的。普通mp4用于点播场景，fmp4通常用于直播场景。

它们有以下差别：

- 普通mp4的时长、内容通常是固定的。fMP4 时长、内容通常不固定，可以边生成边播放；
- 普通mp4完整的metadata都在moov里，需要加载完moov box后，才能对mdat中的媒体数据进行解码渲染；
- fMP4中，媒体数据的metadata在moof box中，moof 跟 mdat （通常）结对出现。moof 中包含了sample duration、sample size等信息，因此，fMP4可以边生成边播放；

## mvex（Movie Extends Box）
当存在mvex时，表示当前文件是fmp4（非严谨）。此时，sample相关的metadata不在moov里，需要通过解析moof box来获得。

### mehd（Movie Extends Header Box）
mehd是可选的，用来声明影片的完整时长（fragment_duration）。如果不存在，则需要遍历所有的fragment，来获得完整的时长。对于fmp4的场景，fragment_duration一般没办法提前预知。

### trex（Track Extends Box）

用来给 fMP4 的 sample 设置各种默认值，比如时长、大小等。

## moof（Movie Fragment Box）
moof是个container box，相关 metadata 在内嵌box里，比如 mfhd、 tfhd、trun 等。
![[Pasted image 20230809205853.png]]
### mfhd（Movie Fragment Header Box）

结构比较简单，sequence_number 为 movie fragment 的序列号。根据 movie fragment 产生的顺序，从1开始递增。

### traf（Track Fragment Box）
对 fmp4 来说，数据被分为多个 movie fragment。一个 movie fragment 可包含多个track fragment（每个 track 包含0或多个 track fragment）。每个 track fragment 中，可以包含多个该 track 的 sample。

### tfhd（Track Fragment Header Box）
tfhd 用来设置 track fragment 中 的 sample 的 metadata 的默认值。

### trun（Track Fragment Run Box）

track run 表示一组连续的 sample，其中：
- sample_count：sample 的数目；
- data_offset：数据部分的偏移量；
- first_sample_flags：可选，针对当前 track run中 第一个 sample 的设置；

