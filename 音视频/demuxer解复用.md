```c++
AVInputFormat ff_mov_demuxer = {
    .name           = "mov,mp4,m4a,3gp,3g2,mj2",
    .long_name      = NULL_IF_CONFIG_SMALL("QuickTime / MOV"),
    .priv_class     = &mov_class,
    .priv_data_size = sizeof(MOVContext),
    .extensions     = "mov,mp4,m4a,3gp,3g2,mj2,psp,m4b,ism,ismv,isma,f4v",
    .read_probe     = mov_probe,
    .read_header    = mov_read_header,
    .read_packet    = mov_read_packet,
    .read_close     = mov_read_close,
    .read_seek      = mov_read_seek,
    .flags          = AVFMT_NO_BYTE_SEEK | AVFMT_SEEK_TO_PTS,
};
```

FFmpeg 的 Demuxer 解复用架构可以分为 3 个部分，以 MP4 为例
![[Pasted image 20230810132711.jpg]]


# 一、avformat_open_input打开输入
`avformat_open_input` 函数的主要作用是打开一个输入源，读取它的头部，然后进行解析。
![[Pasted image 20230810215415.jpg]]

## 1.avformat_alloc_context() 创建容器实例
```c++
# utils.c 536
int avformat_open_input(AVFormatContext **ps, const char *url, ff_const59 AVInputFormat *fmt, AVDictionary **options);
```

函数内会对pb进行判断
```c++
# utils.c 556
if (s->pb) // must be before any goto fail
    s->flags |= AVFMT_FLAG_CUSTOM_IO;
```



## 2.init_input()，初始化输入
`init_input()` 函数的作用是，在未确定输入格式的时候，读取输入文件的一小段内容进行分析探测，进而确定输入文件的格式。

如果命令行里面使用了 `-f xxx` 指定了输入格式，`init_input()` 函数内部就不会进行 `probe` 探测操作，只是会对一些变量进行初始化。

`init_input()` 里面会打开输入文件，或者建立 TCP，RTMP 等网络连接。
```c++
# utils.c 572
if ((ret = init_input(s, filename, &tmp)) < 0)
        goto fail;
s->probe_score = ret;
```

![[Pasted image 20230810222305.jpg]]

上图分为两个条件，用户 是不是 自己定义了 AVIO。如下：

#### 1，用户自己定义了 `AVIO`，也就是自己设置了 `s->pb`
由于 `if (s->pb) {...}` 是在 `avformat_alloc_context()` 之后就执行的，`avformat_alloc_context()` 才刚刚申请了 `AVFormatContext` 的内存，所以通常情况 `s->pb` 都是 `NULL`。

只有用户在 `avformat_open_input()` 之前自己申请了 `AVFormatContext`，并且自己创建了 `AVIOContext`，然后绑定到 `s->pb` 里面，那 `if (s->pb) {...}` 这个条件才会为真。

用户如果已经设置了 `s->iformat`，就不会调 `av_probe_input_buffer2()` 来探测输入的数据，直接就返回了，相当于 `init_input()` 函数什么都没有做。


`av_probe_input_buffer2()` 会遍历所有的 `Demuxer`，调他们的 `probe` 函数来分析输入的数据。

#### 2、用户没有自己定义 AVIO
```c++
if (s->pb) {
	s->flags |= AVFMT_FLAG_CUSTOM_IO;
	if (!s->iformat)
		return av_probe_input_buffer2(s->pb, &s->iformat, filename,
									 s, 0, s->format_probesize);
	else if (s->iformat->flags & AVFMT_NOFILE)
		av_log(s, AV_LOG_WARNING, "Custom AVIOContext makes no sense and "
								  "will be ignored with AVFMT_NOFILE format.\n");
	return 0;
}

if ((s->iformat && s->iformat->flags & AVFMT_NOFILE) ||
	(!s->iformat && (s->iformat = av_probe_input_format2(&pd, 0, &score))))
	return score;

if ((ret = s->io_open(s, &s->pb, filename, AVIO_FLAG_READ | s->avio_flags, options)) < 0)
	return ret;

if (s->iformat)
	return 0;
return av_probe_input_buffer2(s->pb, &s->iformat, filename,
							 s, 0, s->format_probesize);
```
`av_probe_input_format2()` 与 `av_probe_input_buffer2()` 其实都会遍历 一部分的 Dumxer 来探测输入格式。

但是注意，`av_probe_input_buffer2()` 是在 `s->io_open()` 之后执行的。

真相就是，`av_probe_input_format2()` 遍历的是有 `AVFMT_NOFILE` 标记的 `Demuxer`， `av_probe_input_buffer2()` 遍历的是没有 `AVFMT_NOFILE` 标记的 `Demuxer`。

因为如果有 `AVFMT_NOFILE` 标记，是不需要执行 `s->io_open()` 打开输入文件的。
![[Pasted image 20230811141251.png]]

av_probe_input_buffer2()用avio_read()读走了mp4开头的一部分内容到pd，调用mov_probe函数判断这个pd的数据是符合mp4的

### mov_probe
用AV_RB32等函数读取出来 box 的 size 跟 tag,MP4 文件刚开始的 0 ~ 3 字节是 box的 size （大小），然后 4 ~ 7 字节是 box 的类型。

mov_prove函数里面的for (;;) {...}循环，就是在不断遍历AVIOContext的buffer里面的所有box，匹配他们的类型tag，得出分数score。

![[Pasted image 20230812210914.png]]

![[Pasted image 20230812211036.png]]

![[Pasted image 20230812205108.png]]





## 3.对容器的私有数据 s->priv_data 进行赋值
```c++
if (s->iformat->priv_data_size > 0) {
	if (!(s->priv_data = av_mallocz(s->iformat->priv_data_size)))  //申请内存
	{
		ret = AVERROR(ENOMEM);
		goto fail;
	}
	if (s->iformat->priv_class) {
		*(const AVClass **) s->priv_data = s->iformat->priv_class; // 拷贝内存
		av_opt_set_defaults(s->priv_data);
		if ((ret = av_opt_set_dict(s->priv_data, &tmp)) < 0)
			goto fail;
	}
}
```
他不是在进行 **强制类型转换**，而是把 `s->iformat->priv_class` 的内存数据 拷贝到 `s->priv_data`，作用等同于 `memcpy()` 函数。

`s` 变量是 `AVFormatContext`（容器），`AVFormatContext` 是一个通用的数据结构，MP4，FLV 都用这个结构，但是不同的 Demuxer，他们内部的数据结构肯定是不一样的，所以需要在 `AVFormatContext` （通用数据结构）里面绑定一个 `priv_data`（私有数据结构）。

`priv_data` 是一个 `void` 指针，他的内存大小是不固定的，由 各个 `Demuxer` 自己定义，如下：
```c++
AVInputFormat ff_mov_demuxer = {
    .name           = "mov,mp4,m4a,3gp,3g2,mj2",
    .long_name      = NULL_IF_CONFIG_SMALL("QuickTime / MOV"),
    .priv_class     = &mov_class,
    .priv_data_size = sizeof(MOVContext),
    .extensions     = "mov,mp4,m4a,3gp,3g2,mj2,psp,m4b,ism,ismv,isma,f4v",
    .read_probe     = mov_probe,
    .read_header    = mov_read_header,
    .read_packet    = mov_read_packet,
    .read_close     = mov_read_close,
    .read_seek      = mov_read_seek,
    .flags          = AVFMT_NO_BYTE_SEEK | AVFMT_SEEK_TO_PTS,
};
```

在 MP4 Demuxer 里，priv_data 的大小就是 MOVContext 结构的大小。
当进入 MP4 的模块 mov.c 的时候，这块 priv_data 的内存，就会转成 MOVContext 的结构进行使用。

MOVContext是整个 MP4 的管理器，他在ff_mov_demuxer里面有被使用

priv_class指向mov_class，mov_class是一个静态变量来的，mov_class变量的这些静态内存后面会在avformat_open_input()里被拷贝给AVFormatContext的priv_data。

### MOVContext数据结构字段
```c++
typedef struct MOVContext {
    const AVClass *class; ///< class for private options
    AVFormatContext *fc;
    int time_scale;
    int64_t duration;     ///< duration of the longest track
    int found_moov;       ///< 'moov' atom has been found
    int found_mdat;       ///< 'mdat' atom has been found
    int found_hdlr_mdta;  ///< 'hdlr' atom with type 'mdta' has been found
    int trak_index;       ///< Index of the current 'trak'
    char **meta_keys;
    unsigned meta_keys_count;
    DVDemuxContext *dv_demux;
    AVFormatContext *dv_fctx;
    int isom;             ///< 1 if file is ISO Media (mp4/3gp)
    MOVFragment fragment; ///< current fragment in moof atom
    MOVTrackExt *trex_data;
    unsigned trex_count;
    int itunes_metadata;  ///< metadata are itunes style
    int handbrake_version;
    int *chapter_tracks;
    unsigned int nb_chapter_tracks;
    int use_absolute_path;
    int ignore_editlist;
    int advanced_editlist;
    int ignore_chapters;
    int seek_individually;
    int64_t next_root_atom; ///< offset of the next root atom
    int export_all;
    int export_xmp;
    int *bitrates;          ///< bitrates read before streams creation
    int bitrates_count;
    int moov_retry;
    int use_mfra_for;
    int has_looked_for_mfra;
    MOVFragmentIndex frag_index;
    int atom_depth;
    unsigned int aax_mode;  ///< 'aax' file has been detected
    uint8_t file_key[20];
    uint8_t file_iv[20];
    void *activation_bytes;
    int activation_bytes_size;
    void *audible_fixed_key;
    int audible_fixed_key_size;
    struct AVAES *aes_decrypt;
    uint8_t *decryption_key;
    int decryption_key_len;
    int enable_drefs;
    int32_t movie_display_matrix[3][3]; ///< display matrix from mvhd
} MOVContext;
```


#### **1. time_scale，时间基**
这个字段是从 mvhd box 里面读取出来的

#### **2. duration，时长**
这个字段也是从 mvhd box 里面读取出来的


```c++
static int mov_read_mvhd(MOVContext *c, AVIOContext *pb, MOVAtom atom)
{
    int i;
    int64_t creation_time;
    int version = avio_r8(pb); /* version */
    avio_rb24(pb); /* flags */

    if (version == 1) {
        creation_time = avio_rb64(pb);
        avio_rb64(pb);
    } else {
        creation_time = avio_rb32(pb);
        avio_rb32(pb); /* modification time */
    }
    mov_metadata_creation_time(&c->fc->metadata, creation_time, c->fc);
    c->time_scale = avio_rb32(pb); /* time scale */
    if (c->time_scale <= 0) {
        av_log(c->fc, AV_LOG_ERROR, "Invalid mvhd time scale %d, defaulting to 1\n", c->time_scale);
        c->time_scale = 1;
    }
    av_log(c->fc, AV_LOG_TRACE, "time scale = %i\n", c->time_scale);

    c->duration = (version == 1) ? avio_rb64(pb) : avio_rb32(pb); /* duration */
    // set the AVCodecContext duration because the duration of individual tracks
    // may be inaccurate
    if (c->time_scale > 0 && !c->trex_data)
        c->fc->duration = av_rescale(c->duration, AV_TIME_BASE, c->time_scale);
    avio_rb32(pb); /* preferred scale */

    avio_rb16(pb); /* preferred volume */

    avio_skip(pb, 10); /* reserved */

    /* movie display matrix, store it in main context and use it later on */
    for (i = 0; i < 3; i++) {
        c->movie_display_matrix[i][0] = avio_rb32(pb); // 16.16 fixed point
        c->movie_display_matrix[i][1] = avio_rb32(pb); // 16.16 fixed point
        c->movie_display_matrix[i][2] = avio_rb32(pb); //  2.30 fixed point
    }

    avio_rb32(pb); /* preview time */
    avio_rb32(pb); /* preview duration */
    avio_rb32(pb); /* poster time */
    avio_rb32(pb); /* selection time */
    avio_rb32(pb); /* selection duration */
    avio_rb32(pb); /* current time */
    avio_rb32(pb); /* next track ID */

    return 0;
}
```

#### **3. found_moov，found_mdat，found_hdlr_mdta**
MP4 里是否有这些 box,在mov_read_header()的过程中，如果找到了对应的 box，就会把上面这些变量置为 1
```c++
static int mov_read_moov(MOVContext *c, AVIOContext *pb, MOVAtom atom)
{
    int ret;

    if (c->found_moov) {
        av_log(c->fc, AV_LOG_WARNING, "Found duplicated MOOV Atom. Skipped it\n");
        avio_skip(pb, atom.size);
        return 0;
    }

    if ((ret = mov_read_default(c, pb, atom)) < 0)
        return ret;
    /* we parsed the 'moov' atom, we can terminate the parsing as soon as we find the 'mdat' */
    /* so we don't parse the whole file if over a network */
    c->found_moov=1;
    return 0; /* now go for mdat */
}
```
#### 4.trak_index，当前的流索引
trak_index可以说是用来统计mov_read_header()里发现了几个数据流，就是在avformat_new_stream()之后，就会直接把trak_index递增了
```c++
static int mov_read_trak(MOVContext *c, AVIOContext *pb, MOVAtom atom)
{
    AVStream *st;
    MOVStreamContext *sc;
    int ret;

    st = avformat_new_stream(c->fc, NULL);
    if (!st) return AVERROR(ENOMEM);
    st->id = -1;
    sc = av_mallocz(sizeof(MOVStreamContext));
    if (!sc) return AVERROR(ENOMEM);

    st->priv_data = sc;
    st->codecpar->codec_type = AVMEDIA_TYPE_DATA;
    sc->ffindex = st->index;
    c->trak_index = st->index;
}
```

#### 5.moov_retry，查找两次 moov box
当第一次mov_read_default()调用找不到moov，就会调avio_seek()跳转到文件开头，再找一次

#### 6.int atom_depth;
atom_depth变量是用来解析 Box 的深度的，因为mov_read_default()是一个嵌套调用的函数，如下：
![[Pasted image 20230812214714.png]]
atom_depth就是用来记录当前解析到第几层的 Box 了，如下，每次进入mov_read_default()，atom_depth变量都会加 1。

### MOVStreamContext数据结构
MOVStreamContext实际上是 MP4 里面的流管理器，MP4 里面可以有多个视频流，多个音频流的，他们就是 trak Box
MOVStreamContext是 trak Box 以及他下面的 Box 的管理器。
```C++
typedef struct MOVStreamContext {
    AVIOContext *pb;
    int pb_is_copied;
    int ffindex;          ///< AVStream index
    int next_chunk;
    unsigned int chunk_count;
    int64_t *chunk_offsets;
    unsigned int stts_count;
    MOVStts *stts_data;
    unsigned int sdtp_count;
    uint8_t *sdtp_data;
    unsigned int ctts_count;
    unsigned int ctts_allocated_size;
    MOVStts *ctts_data;
    unsigned int stsc_count;
    MOVStsc *stsc_data;
    unsigned int stsc_index;
    int stsc_sample;
    unsigned int stps_count;
    unsigned *stps_data;  ///< partial sync sample for mpeg-2 open gop
    MOVElst *elst_data;
    unsigned int elst_count;
    int ctts_index;
    int ctts_sample;
    unsigned int sample_size; ///< may contain value calculated from stsd or value from stsz atom
    unsigned int stsz_sample_size; ///< always contains sample size from stsz atom
    unsigned int sample_count;
    int *sample_sizes;
    int keyframe_absent;
    unsigned int keyframe_count;
    int *keyframes;
    int time_scale;
    int64_t time_offset;  ///< time offset of the edit list entries
    int64_t min_corrected_pts;  ///< minimum Composition time shown by the edits excluding empty edits.
    int current_sample;
    int64_t current_index;
    MOVIndexRange* index_ranges;
    MOVIndexRange* current_index_range;
    unsigned int bytes_per_frame;
    unsigned int samples_per_frame;
    int dv_audio_container;
    int pseudo_stream_id; ///< -1 means demux all ids
    int16_t audio_cid;    ///< stsd audio compression id
    unsigned drefs_count;
    MOVDref *drefs;
    int dref_id;
    int timecode_track;
    int width;            ///< tkhd width
    int height;           ///< tkhd height
    int dts_shift;        ///< dts shift when ctts is negative
    uint32_t palette[256];
    int has_palette;
    int64_t data_size;
    uint32_t tmcd_flags;  ///< tmcd track flags
    int64_t track_end;    ///< used for dts generation in fragmented movie files
    int start_pad;        ///< amount of samples to skip due to enc-dec delay
    unsigned int rap_group_count;
    MOVSbgp *rap_group;

    int nb_frames_for_fps;
    int64_t duration_for_fps;

    /** extradata array (and size) for multiple stsd */
    uint8_t **extradata;
    int *extradata_size;
    int last_stsd_index;
    int stsd_count;
    int stsd_version;

    int32_t *display_matrix;
    AVStereo3D *stereo3d;
    AVSphericalMapping *spherical;
    size_t spherical_size;
    AVMasteringDisplayMetadata *mastering;
    AVContentLightMetadata *coll;
    size_t coll_size;

    uint32_t format;

    int has_sidx;  // If there is an sidx entry for this stream.
    struct {
        struct AVAESCTR* aes_ctr;
        unsigned int per_sample_iv_size;  // Either 0, 8, or 16.
        AVEncryptionInfo *default_encrypted_sample;
        MOVEncryptionIndex *encryption_index;
    } cenc;
} MOVStreamContext;

```

#### 1.stco 表的chunk信息，chunk_count、chunk_offsets
Stco这个表记录的是每个chunk在文件中的位置

一个chunk里可以只有一个视频帧，也可以有多个视频帧

在isom.h里面是没有定义MOVStco数据结构的，他用的是chunk_count跟chunk_offsets字段来存储stco表的数据
```c++
unsigned int chunk_count;
int64_t *chunk_offsets;
```
chunk_count、chunk_offsets是在mov_read_stco()里面进行赋值的
```c++
sc->chunk_count = 0;
sc->chunk_offsets = av_malloc_array(entries, sizeof(*sc->chunk_offsets));
if (!sc->chunk_offsets)
	return AVERROR(ENOMEM);
sc->chunk_count = entries;

if      (atom.type == MKTAG('s','t','c','o'))
	for (i = 0; i < entries && !pb->eof_reached; i++)
		sc->chunk_offsets[i] = avio_rb32(pb);
else if (atom.type == MKTAG('c','o','6','4'))
	for (i = 0; i < entries && !pb->eof_reached; i++)
		sc->chunk_offsets[i] = avio_rb64(pb);
else
	return AVERROR_INVALIDDATA;

sc->chunk_count = i;
```
把 mp4 里面的stco表的数据读取到chunk_offsets数组

  
#### 2.stsc *stsc_data，chunk 与 Sample 的映射表
  
一个chunk里面可能会有多个sample,stsc表记录的就是chunk与Sample之间的关系

isom.h里面跟stsc表相关的数据结构如下:
```c++
unsigned int stsc_count;
MOVStsc *stsc_data;
unsigned int stsc_index;
int stsc_sample;
```
stsc_count跟stsc_data实际上就是原封不动地读取的 mp4 文件的stsc 表的数据的。

因为 stsc 表的数据是压缩后的数据，如果多行映射信息是一样的，他就会只记录第一行的映射数据，这就是压缩。所以在使用 stsc 表的数据的时候需要进行解压，stsc_index与stsc_sample就是实现解压功能的
```c++
/* Multiple stsd handling. */
if (sc->stsc_data) {
	/* Keep track of the stsc index for the given sample, then check
	* if the stsd index is different from the last used one. */
	sc->stsc_sample++;
	if (mov_stsc_index_valid(sc->stsc_index, sc->stsc_count) &&
		mov_get_stsc_samples(sc, sc->stsc_index) == sc->stsc_sample) {
		sc->stsc_index++;
		sc->stsc_sample = 0;
	/* Do not check indexes after a switch. */
	} else if (sc->stsc_data[sc->stsc_index].id > 0 &&
			   sc->stsc_data[sc->stsc_index].id - 1 < sc->stsd_count &&
			   sc->stsc_data[sc->stsc_index].id - 1 != sc->last_stsd_index) {
		ret = mov_change_extradata(sc, pkt);
		if (ret < 0)
			return ret;
	}
}
```

在 MP4 里面，解压是非常常见的，**一旦你简单变量是xxx_index与xxx_sample成对出现的，那这些变量大概率就是用来进行解压的。**


#### 3.stsz 表， Sample（帧）大小信息
stsz表存储的是每一帧数据的大小，在isom.h里面是没有定义MOVStsz数据结构的，他用的是sample_count跟sample_sizes字段来存储stsz表的数据
```c++
unsigned int sample_count;
int *sample_sizes;
```

#### 4.MOVStts *stts_data，解码时间表*
stts表存储的是每一帧的解码时间，在没有 B 帧的情况下，解码时间就等于播放时间了。stts表存储的是压缩后的数据

```c++
typedef struct MOVStts { 
	unsigned int count; 
	int duration;
} MOVStts;
```
mov.c的mov_read_stts()函数实际上是把 stts 表的数据原封不动读取到stts_data字段里面，是没有进行解压的
```c++
sc->stts_data[i].count= sample_count;
sc->stts_data[i].duration= sample_duration;
```

sc->stts_data[]的数据最后会解压，然后把每一帧的解码时间放到index_entries[]里
mov_read_stts()函数里面也会利用stts表的数据来计算AVStream的duration，如下
```c++
duration+=(int64_t)sample_duration*(uint64_t)sample_count;
total_sample_count+=sample_count;

if (duration)
    st->duration= FFMIN(st->duration, duration);
```

#### 5.MOVStts *ctts_data，时间补偿表*
之前的stts表记录了每一帧的解码时间，**如果有 B 帧，解码时间跟播放时间是不一样的**。这时候为了计算出播放时间，就需要用到时间补偿表ctts。

```c++
int ctts_index;
int ctts_sample;
```
ctts_index代表当前使用到 ctts 表的第几行的数据，ctts 表的一行可能有多个帧的，因为多个帧的补偿时间一样，所以被压缩成一行了。ctts_sample就代表现在已经遍历到了 ctts 表的某一行的第几帧。这两个变量实际上就是实现解压功能的
```c++
pkt->stream_index = sc->ffindex;
pkt->dts = sample->timestamp;
if (sample->flags & AVINDEX_DISCARD_FRAME) {
	pkt->flags |= AV_PKT_FLAG_DISCARD;
}
if (sc->ctts_data && sc->ctts_index < sc->ctts_count) {
	pkt->pts = pkt->dts + sc->dts_shift + sc->ctts_data[sc->ctts_index].duration;
	/* update ctts context */
	sc->ctts_sample++;  // 当前行的帧数不断增加
	if (sc->ctts_index < sc->ctts_count &&
		sc->ctts_data[sc->ctts_index].count == sc->ctts_sample) {
		sc->ctts_index++;
		sc->ctts_sample = 0;  // 进入ctts下一行时清空ctts_sample
	}
} else {
	int64_t next_dts = (sc->current_sample < st->nb_index_entries) ?
		st->index_entries[sc->current_sample].timestamp : st->duration;

	if (next_dts >= pkt->dts)
		pkt->duration = next_dts - pkt->dts;
	pkt->pts = pkt->dts;
}
```


  
#### 6.stss，关键帧表
在isom.h里面是没有定义MOVStss数据结构的，他用的是keyframe_count跟keyframes字段来存储stss表的数据
```c++
for (i = 0; i < entries && !pb->eof_reached; i++) {
    sc->keyframes[i] = avio_rb32(pb);
}
sc->keyframe_count = i;
```

#### 7.time_scale，时间基
time_scale变量的值是从mdhd里面读取出来的
```c++
sc->time_scale = avio_rb32(pb);
if (sc->time_scale <= 0) {
	av_log(c->fc, AV_LOG_ERROR, "Invalid mdhd time scale %d, defaulting to 1\n", sc->time_scale);
	sc->time_scale = 1;
}
```

stts表里面的 Sample delta 就是用的time_scale时间基础


#### 7.stsd表解析
stsd 的全称是Sample Description Box，采样描述表。主要存储的是一些编码信息
![[Pasted image 20230813011943.png]]
在mov.c里面，有两个函数是专门负责解析stsd表的，mov_parse_stsd_video()与mov_parse_stsd_audio()


## 4.s->iformat->read_header(s) 读取头部，进行解析
`read_header` 是一个函数指针，指向 Demuxer 的 `read_header` 指针，如果 是 MP4，那 `read_header` 就指向 `mov_read_header` 函数，如下：
```c++
.read_header    = mov_read_header,
```

`mov_read_header` 在函数调用中的位置如下：
![[Pasted image 20230813012722.jpg]]

`mov_read_header()` 函数内部的主要流程如下：
![[Pasted image 20230813012830.jpg]]

`mov_read_header()` 这个函数名称，我个人觉得 是有些许误导的作用的，其实 MP4 格式没有 header 头部一说，他的结构全是 Box。

所以 `mov_read_header()` 函数最主要的工作，就是==调 `mov_read_default()` 解析所有的 Box==，**把他们的基本信息存储到 `MOVContext` 跟 `MOVStreamContext` 里面。**

`mov_read_header()` 函数有以下重点：

#### 1，MOVContext 类型转换

```c++
MOVContext *mov = s->priv_data;
AVIOContext *pb = s->pb;
```
`priv_data` 是绑定在 `AVFormatContext` （通用数据结构）里面的私有内存字段，所以一旦进入 `mov.c` 这种私有的模块，就需要把 `priv_data` 转成 `MOVContext`（私有数据结构）来使用，这里 `MOVContext` 就是 MP4 的私有数据结构。

`priv_data` 变量的内存数据，是之前在 `avformat_open_input()` ➔ `init_input()` 里赋值的，如下：
![[Pasted image 20230813013909.png]]
而 `s->iforamt` 实际上就是 `ff_mov_demuxer`，如下：
![[Pasted image 20230813013916.png]]

#### 2.isom.h 头文件

MP4 格式的数据结构都定义在 `isom.h` 里面的，isom 的全称是 ISO Base Media file，是在 MPEG-4 Part 12 中定义的一种基础文件格式。MP4 文件格式就是基于 isom 衍生出来的。

#### 3，MOVAtom 数据结构
```c++
typedef struct MOVAtom {
    uint32_t type;
    int64_t size; /* total size (excluding the size and type fields) */
} MOVAtom;
```

```c++
//mov_read_header函数的第 4 行代码
MOVAtom atom = { AV_RL32("root") };
```
`MOVAtom` 数据结构可以看成是 MP4 格式中的 box。mp4 格式里，Box 的前面 0~3 字节是 size （大小），然后 4~7 字节是 box 的类型，后面如果还有数据，就是 Box 的具体的数据。

#### 4，mov_read_default，从 root 顶部，解析底下所有的 Box
```c++
/* check MOV header */

do {
	if (mov->moov_retry)
		avio_seek(pb, 0, SEEK_SET);
	if ((err = mov_read_default(mov, pb, atom)) < 0) {
		av_log(s, AV_LOG_ERROR, "error reading header\n");
		goto fail;
	}
} while ((pb->seekable & AVIO_SEEKABLE_NORMAL) && !mov->found_moov && !mov->moov_retry++);
```

`mov_read_default` 可以解析任何一种类型的 Box 的，不只是可以从 顶部 root 开始解析。还可以从 moov Box 开始解析;
```c++
static int mov_read_moov(MOVContext *c, AVIOContext *pb, MOVAtom atom)
{
    int ret;

    if (c->found_moov) {
        av_log(c->fc, AV_LOG_WARNING, "Found duplicated MOOV Atom. Skipped it\n");
        avio_skip(pb, atom.size);
        return 0;
    }

    if ((ret = mov_read_default(c, pb, atom)) < 0)
        return ret;
    /* we parsed the 'moov' atom, we can terminate the parsing as soon as we find the 'mdat' */
    /* so we don't parse the whole file if over a network */
    c->found_moov=1;
    return 0; /* now go for mdat */
}
```

`mov_read_default()` 是一个可以多层调用的函数，如果有多层 Box 是一直嵌套下去解析的。每个 Box 对应的解析方法是定义在 `mov_default_parse_table[]` 数组里面的，如下：
```c++
static const MOVParseTableEntry mov_default_parse_table[] = {
{ MKTAG('A','C','L','R'), mov_read_aclr },
{ MKTAG('A','P','R','G'), mov_read_avid },
{ MKTAG('A','A','L','P'), mov_read_avid },
{ MKTAG('A','R','E','S'), mov_read_ares },
{ MKTAG('a','v','s','s'), mov_read_avss },
{ MKTAG('a','v','1','C'), mov_read_av1c },
{ MKTAG('c','h','p','l'), mov_read_chpl },
{ MKTAG('c','o','6','4'), mov_read_stco },
{ MKTAG('c','o','l','r'), mov_read_colr },
{ MKTAG('c','t','t','s'), mov_read_ctts }, /* composition time to sample */
{ MKTAG('d','i','n','f'), mov_read_default },
{ MKTAG('D','p','x','E'), mov_read_dpxe },
{ MKTAG('d','r','e','f'), mov_read_dref },
{ MKTAG('e','d','t','s'), mov_read_default },
{ MKTAG('e','l','s','t'), mov_read_elst },
{ MKTAG('e','n','d','a'), mov_read_enda },
{ MKTAG('f','i','e','l'), mov_read_fiel },
{ MKTAG('a','d','r','m'), mov_read_adrm },
{ MKTAG('f','t','y','p'), mov_read_ftyp },
{ MKTAG('g','l','b','l'), mov_read_glbl },
{ MKTAG('h','d','l','r'), mov_read_hdlr },
{ MKTAG('i','l','s','t'), mov_read_ilst },
{ MKTAG('j','p','2','h'), mov_read_jp2h },
{ MKTAG('m','d','a','t'), mov_read_mdat },
{ MKTAG('m','d','h','d'), mov_read_mdhd },
{ MKTAG('m','d','i','a'), mov_read_default },
{ MKTAG('m','e','t','a'), mov_read_meta },
{ MKTAG('m','i','n','f'), mov_read_default },
{ MKTAG('m','o','o','f'), mov_read_moof },
{ MKTAG('m','o','o','v'), mov_read_moov },
{ MKTAG('m','v','e','x'), mov_read_default },
{ MKTAG('m','v','h','d'), mov_read_mvhd },
{ MKTAG('S','M','I',' '), mov_read_svq3 },
{ MKTAG('a','l','a','c'), mov_read_alac }, /* alac specific atom */
{ MKTAG('a','v','c','C'), mov_read_glbl },
{ MKTAG('p','a','s','p'), mov_read_pasp },
{ MKTAG('s','i','d','x'), mov_read_sidx },
{ MKTAG('s','t','b','l'), mov_read_default },
{ MKTAG('s','t','c','o'), mov_read_stco },
{ MKTAG('s','t','p','s'), mov_read_stps },
{ MKTAG('s','t','r','f'), mov_read_strf },
{ MKTAG('s','t','s','c'), mov_read_stsc },
{ MKTAG('s','t','s','d'), mov_read_stsd }, /* sample description */
{ MKTAG('s','t','s','s'), mov_read_stss }, /* sync sample */
{ MKTAG('s','t','s','z'), mov_read_stsz }, /* sample size */
{ MKTAG('s','t','t','s'), mov_read_stts },
{ MKTAG('s','t','z','2'), mov_read_stsz }, /* compact sample size */
{ MKTAG('s','d','t','p'), mov_read_sdtp }, /* independent and disposable samples */
{ MKTAG('t','k','h','d'), mov_read_tkhd }, /* track header */
{ MKTAG('t','f','d','t'), mov_read_tfdt },
{ MKTAG('t','f','h','d'), mov_read_tfhd }, /* track fragment header */
{ MKTAG('t','r','a','k'), mov_read_trak },
{ MKTAG('t','r','a','f'), mov_read_default },
{ MKTAG('t','r','e','f'), mov_read_default },
{ MKTAG('t','m','c','d'), mov_read_tmcd },
{ MKTAG('c','h','a','p'), mov_read_chap },
{ MKTAG('t','r','e','x'), mov_read_trex },
{ MKTAG('t','r','u','n'), mov_read_trun },
{ MKTAG('u','d','t','a'), mov_read_default },
{ MKTAG('w','a','v','e'), mov_read_wave },
{ MKTAG('e','s','d','s'), mov_read_esds },
{ MKTAG('d','a','c','3'), mov_read_dac3 }, /* AC-3 info */
{ MKTAG('d','e','c','3'), mov_read_dec3 }, /* EAC-3 info */
{ MKTAG('d','d','t','s'), mov_read_ddts }, /* DTS audio descriptor */
{ MKTAG('w','i','d','e'), mov_read_wide }, /* place holder */
{ MKTAG('w','f','e','x'), mov_read_wfex },
{ MKTAG('c','m','o','v'), mov_read_cmov },
{ MKTAG('c','h','a','n'), mov_read_chan }, /* channel layout */
{ MKTAG('d','v','c','1'), mov_read_dvc1 },
{ MKTAG('s','b','g','p'), mov_read_sbgp },
{ MKTAG('h','v','c','C'), mov_read_glbl },
{ MKTAG('u','u','i','d'), mov_read_uuid },
{ MKTAG('C','i','n', 0x8e), mov_read_targa_y216 },
{ MKTAG('f','r','e','e'), mov_read_free },
{ MKTAG('-','-','-','-'), mov_read_custom },
{ MKTAG('s','i','n','f'), mov_read_default },
{ MKTAG('f','r','m','a'), mov_read_frma },
{ MKTAG('s','e','n','c'), mov_read_senc },
{ MKTAG('s','a','i','z'), mov_read_saiz },
{ MKTAG('s','a','i','o'), mov_read_saio },
{ MKTAG('p','s','s','h'), mov_read_pssh },
{ MKTAG('s','c','h','m'), mov_read_schm },
{ MKTAG('s','c','h','i'), mov_read_default },
{ MKTAG('t','e','n','c'), mov_read_tenc },
{ MKTAG('d','f','L','a'), mov_read_dfla },
{ MKTAG('s','t','3','d'), mov_read_st3d }, /* stereoscopic 3D video box */
{ MKTAG('s','v','3','d'), mov_read_sv3d }, /* spherical video box */
{ MKTAG('d','O','p','s'), mov_read_dops },
{ MKTAG('d','m','l','p'), mov_read_dmlp },
{ MKTAG('S','m','D','m'), mov_read_smdm },
{ MKTAG('C','o','L','L'), mov_read_coll },
{ MKTAG('v','p','c','C'), mov_read_vpcc },
{ MKTAG('m','d','c','v'), mov_read_mdcv },
{ MKTAG('c','l','l','i'), mov_read_clli },
{ MKTAG('d','v','c','C'), mov_read_dvcc_dvvc },
{ MKTAG('d','v','v','C'), mov_read_dvcc_dvvc },
{ 0, NULL }
};
```

上面这些方法会解析 Box 的基本信息，然后存储到 `MOVContext` 或者 `MOVStreamContext` 对象。

```c++
for (i = 0; mov_default_parse_table[i].type; i++)
    if (mov_default_parse_table[i].type == a.type) {
        parse = mov_default_parse_table[i].parse;
        break;
    }
    ...省略代码...
}
```

例如 `duration` 文件的时长，就是在 `mov_read_mvhd()` 里面解析出来的

而 `AVStream` 流对象是在 `mov_read_trak()` 里面创建的
```c++
st = avformat_new_stream(c->fc, NULL);

if ((ret = mov_read_default(c, pb, atom)) < 0)
        return ret;
```
`trak` 的全称是 track，代表轨道，流的意思。`avformat_new_stream()` 执行完之后，流数量就会 +1

**`mov_read_default()` 函数就是解析各个 Box 的基本信息的，如果 Box 里面还有 Box，就会继续调 `mov_read_default()` 来进行解析。**

#### 5，ff_rfps_calculate，计算真正的帧率

```c++
static int mov_read_header(AVFormatContext *s)
{
	ff_rfps_calculate(s);
}
```

#### 6.mov_read_close，释放资源，释放内存
因为之前在解析所有 Box 的时候，会申请一些数据内存，所以在最后需要把他们释放


## 5.mv_read_trak()
`mov_read_trak()` 被调用的位置如下：
![[Pasted image 20230813025252.png]]

`mov_read_trak()` 函数内部的流程图如下：
![[Pasted image 20230813133806.jpg]]

```c++
st = avformat_new_stream(c->fc, NULL);

sc = av_mallocz(sizeof(MOVStreamContext));

if ((ret = mov_read_default(c, pb, atom)) < 0)
    return ret;

fix_timescale(c, sc);

avpriv_set_pts_info(st, 64, 1, sc->time_scale);

mov_build_index(c, st);
```

#### avformat_new_stream创建数据流
`avformat_new_stream()` 是 FFmpeg 的通用函数，不论是 `mp4`，`flv`，还是 `rtmp` 等等，都是通过 `avformat_new_stream()` 来创建流对象（AVStream）的

大部分情况，`AVStream` 数据流对象 都是在 `avformat_open_input()` 里创建的

##### AVStreamInternal，AVStream 内部使用的数据结构
```c++
/**
 * Stream structure.
 * New fields can be added to the end with minor version bumps.
 * Removal, reordering and changes to existing fields require a major
 * version bump.
 * sizeof(AVStream) must not be used outside libav*.
 */
typedef struct AVStream {
    int index;    /**< stream index in AVFormatContext */
    /**
     * Format-specific stream ID.
     * decoding: set by libavformat
     * encoding: set by the user, replaced by libavformat if left unset
     */
    int id;
#if FF_API_LAVF_AVCTX
    /**
     * @deprecated use the codecpar struct instead
     */
    attribute_deprecated
    AVCodecContext *codec;
#endif
    void *priv_data;

    /**
     * This is the fundamental unit of time (in seconds) in terms
     * of which frame timestamps are represented.
     *
     * decoding: set by libavformat
     * encoding: May be set by the caller before avformat_write_header() to
     *           provide a hint to the muxer about the desired timebase. In
     *           avformat_write_header(), the muxer will overwrite this field
     *           with the timebase that will actually be used for the timestamps
     *           written into the file (which may or may not be related to the
     *           user-provided one, depending on the format).
     */
    AVRational time_base;

    /**
     * Decoding: pts of the first frame of the stream in presentation order, in stream time base.
     * Only set this if you are absolutely 100% sure that the value you set
     * it to really is the pts of the first frame.
     * This may be undefined (AV_NOPTS_VALUE).
     * @note The ASF header does NOT contain a correct start_time the ASF
     * demuxer must NOT set this.
     */
    int64_t start_time;

    /**
     * Decoding: duration of the stream, in stream time base.
     * If a source file does not specify a duration, but does specify
     * a bitrate, this value will be estimated from bitrate and file size.
     *
     * Encoding: May be set by the caller before avformat_write_header() to
     * provide a hint to the muxer about the estimated duration.
     */
    int64_t duration;

    int64_t nb_frames;                 ///< number of frames in this stream if known or 0

    int disposition; /**< AV_DISPOSITION_* bit field */

    enum AVDiscard discard; ///< Selects which packets can be discarded at will and do not need to be demuxed.

    /**
     * sample aspect ratio (0 if unknown)
     * - encoding: Set by user.
     * - decoding: Set by libavformat.
     */
    AVRational sample_aspect_ratio;

    AVDictionary *metadata;

    /**
     * Average framerate
     *
     * - demuxing: May be set by libavformat when creating the stream or in
     *             avformat_find_stream_info().
     * - muxing: May be set by the caller before avformat_write_header().
     */
    AVRational avg_frame_rate;

    /**
     * For streams with AV_DISPOSITION_ATTACHED_PIC disposition, this packet
     * will contain the attached picture.
     *
     * decoding: set by libavformat, must not be modified by the caller.
     * encoding: unused
     */
    AVPacket attached_pic;

    /**
     * An array of side data that applies to the whole stream (i.e. the
     * container does not allow it to change between packets).
     *
     * There may be no overlap between the side data in this array and side data
     * in the packets. I.e. a given side data is either exported by the muxer
     * (demuxing) / set by the caller (muxing) in this array, then it never
     * appears in the packets, or the side data is exported / sent through
     * the packets (always in the first packet where the value becomes known or
     * changes), then it does not appear in this array.
     *
     * - demuxing: Set by libavformat when the stream is created.
     * - muxing: May be set by the caller before avformat_write_header().
     *
     * Freed by libavformat in avformat_free_context().
     *
     * @see av_format_inject_global_side_data()
     */
    AVPacketSideData *side_data;
    /**
     * The number of elements in the AVStream.side_data array.
     */
    int            nb_side_data;

    /**
     * Flags for the user to detect events happening on the stream. Flags must
     * be cleared by the user once the event has been handled.
     * A combination of AVSTREAM_EVENT_FLAG_*.
     */
    int event_flags;
#define AVSTREAM_EVENT_FLAG_METADATA_UPDATED 0x0001 ///< The call resulted in updated metadata.

    /**
     * Real base framerate of the stream.
     * This is the lowest framerate with which all timestamps can be
     * represented accurately (it is the least common multiple of all
     * framerates in the stream). Note, this value is just a guess!
     * For example, if the time base is 1/90000 and all frames have either
     * approximately 3600 or 1800 timer ticks, then r_frame_rate will be 50/1.
     */
    AVRational r_frame_rate;

#if FF_API_LAVF_FFSERVER
    /**
     * String containing pairs of key and values describing recommended encoder configuration.
     * Pairs are separated by ','.
     * Keys are separated from values by '='.
     *
     * @deprecated unused
     */
    attribute_deprecated
    char *recommended_encoder_configuration;
#endif

    /**
     * Codec parameters associated with this stream. Allocated and freed by
     * libavformat in avformat_new_stream() and avformat_free_context()
     * respectively.
     *
     * - demuxing: filled by libavformat on stream creation or in
     *             avformat_find_stream_info()
     * - muxing: filled by the caller before avformat_write_header()
     */
    AVCodecParameters *codecpar;

    /*****************************************************************
     * All fields below this line are not part of the public API. They
     * may not be used outside of libavformat and can be changed and
     * removed at will.
     * Internal note: be aware that physically removing these fields
     * will break ABI. Replace removed fields with dummy fields, and
     * add new fields to AVStreamInternal.
     *****************************************************************
     */

#define MAX_STD_TIMEBASES (30*12+30+3+6)
    /**
     * Stream information used internally by avformat_find_stream_info()
     */
    struct {
        int64_t last_dts;
        int64_t duration_gcd;
        int duration_count;
        int64_t rfps_duration_sum;
        double (*duration_error)[2][MAX_STD_TIMEBASES];
        int64_t codec_info_duration;
        int64_t codec_info_duration_fields;
        int frame_delay_evidence;

        /**
         * 0  -> decoder has not been searched for yet.
         * >0 -> decoder found
         * <0 -> decoder with codec_id == -found_decoder has not been found
         */
        int found_decoder;

        int64_t last_duration;

        /**
         * Those are used for average framerate estimation.
         */
        int64_t fps_first_dts;
        int     fps_first_dts_idx;
        int64_t fps_last_dts;
        int     fps_last_dts_idx;

    } *info;

    int pts_wrap_bits; /**< number of bits in pts (used for wrapping control) */

    // Timestamp generation support:
    /**
     * Timestamp corresponding to the last dts sync point.
     *
     * Initialized when AVCodecParserContext.dts_sync_point >= 0 and
     * a DTS is received from the underlying container. Otherwise set to
     * AV_NOPTS_VALUE by default.
     */
    int64_t first_dts;
    int64_t cur_dts;
    int64_t last_IP_pts;
    int last_IP_duration;

    /**
     * Number of packets to buffer for codec probing
     */
    int probe_packets;

    /**
     * Number of frames that have been demuxed during avformat_find_stream_info()
     */
    int codec_info_nb_frames;

    /* av_read_frame() support */
    enum AVStreamParseType need_parsing;
    struct AVCodecParserContext *parser;

    /**
     * last packet in packet_buffer for this stream when muxing.
     */
    struct AVPacketList *last_in_packet_buffer;
    AVProbeData probe_data;
#define MAX_REORDER_DELAY 16
    int64_t pts_buffer[MAX_REORDER_DELAY+1];

    AVIndexEntry *index_entries; /**< Only used if the format does not
                                    support seeking natively. */
    int nb_index_entries;
    unsigned int index_entries_allocated_size;

    /**
     * Stream Identifier
     * This is the MPEG-TS stream identifier +1
     * 0 means unknown
     */
    int stream_identifier;

    /**
     * Details of the MPEG-TS program which created this stream.
     */
    int program_num;
    int pmt_version;
    int pmt_stream_idx;

    int64_t interleaver_chunk_size;
    int64_t interleaver_chunk_duration;

    /**
     * stream probing state
     * -1   -> probing finished
     *  0   -> no probing requested
     * rest -> perform probing with request_probe being the minimum score to accept.
     */
    int request_probe;
    /**
     * Indicates that everything up to the next keyframe
     * should be discarded.
     */
    int skip_to_keyframe;

    /**
     * Number of samples to skip at the start of the frame decoded from the next packet.
     */
    int skip_samples;

    /**
     * If not 0, the number of samples that should be skipped from the start of
     * the stream (the samples are removed from packets with pts==0, which also
     * assumes negative timestamps do not happen).
     * Intended for use with formats such as mp3 with ad-hoc gapless audio
     * support.
     */
    int64_t start_skip_samples;

    /**
     * If not 0, the first audio sample that should be discarded from the stream.
     * This is broken by design (needs global sample count), but can't be
     * avoided for broken by design formats such as mp3 with ad-hoc gapless
     * audio support.
     */
    int64_t first_discard_sample;

    /**
     * The sample after last sample that is intended to be discarded after
     * first_discard_sample. Works on frame boundaries only. Used to prevent
     * early EOF if the gapless info is broken (considered concatenated mp3s).
     */
    int64_t last_discard_sample;

    /**
     * Number of internally decoded frames, used internally in libavformat, do not access
     * its lifetime differs from info which is why it is not in that structure.
     */
    int nb_decoded_frames;

    /**
     * Timestamp offset added to timestamps before muxing
     */
    int64_t mux_ts_offset;

    /**
     * Internal data to check for wrapping of the time stamp
     */
    int64_t pts_wrap_reference;

    /**
     * Options for behavior, when a wrap is detected.
     *
     * Defined by AV_PTS_WRAP_ values.
     *
     * If correction is enabled, there are two possibilities:
     * If the first time stamp is near the wrap point, the wrap offset
     * will be subtracted, which will create negative time stamps.
     * Otherwise the offset will be added.
     */
    int pts_wrap_behavior;

    /**
     * Internal data to prevent doing update_initial_durations() twice
     */
    int update_initial_durations_done;

    /**
     * Internal data to generate dts from pts
     */
    int64_t pts_reorder_error[MAX_REORDER_DELAY+1];
    uint8_t pts_reorder_error_count[MAX_REORDER_DELAY+1];

    /**
     * Internal data to analyze DTS and detect faulty mpeg streams
     */
    int64_t last_dts_for_order_check;
    uint8_t dts_ordered;
    uint8_t dts_misordered;

    /**
     * Internal data to inject global side data
     */
    int inject_global_side_data;

    /**
     * display aspect ratio (0 if unknown)
     * - encoding: unused
     * - decoding: Set by libavformat to calculate sample_aspect_ratio internally
     */
    AVRational display_aspect_ratio;

    /**
     * An opaque field for libavformat internal usage.
     * Must not be accessed in any way by callers.
     */
    AVStreamInternal *internal;
} AVStream;

```

#### 申请解码器实例内存

```c++
st->internal->avctx = avcodec_alloc_context3(NULL);
```


#### avcodec_parameters_alloc() 函数
```c++
st->codecpar = avcodec_parameters_alloc();
```

#### avpriv_set_pts_info()
`avpriv_set_pts_info()` 函数里面有一个 `pts_wrap_bits` ，这个变量在 mp4 里面是 64。

`pts_wrap_bits` 是指 pts 的回环位数，用来实现回环策略的，例如 第 2 帧的 pts 是 2^64，第三帧超过 64 位最大值，所以第三帧的 pts 是 3，在回环策略的比较下， 3 其实是 大于 2^64 的。

#### mov_build_index(c, st)
`mov_build_index()`函数的主要作用是创建一个数组 `AVIndexEntry[]`，这个数组里记录了每一帧数据的信息
```c++
typedef struct AVIndexEntry {
    int64_t pos; 
    int64_t timestamp;
#define AVINDEX_KEYFRAME 0x0001
#define AVINDEX_DISCARD_FRAME  0x0002    
    int flags:2; 
    int size:30;
    int min_distance; 
} AVIndexEntry;
```

##### pos，一帧数据在文件中的位置
`pos` 这个字段记录的是 这一帧数据在文件中的位置。是通过 `stco` 表的记录算出来的。

不过需要注意 `stco` 记录的是 `chunk` 的位置，而不是 帧的位置，一个 `chunk` 可以有多个帧，所以他在循环里面如果发现有多个帧，会把 `current_offset` 加上 上一帧的大小，这样，`current_offset` 就是当前帧的位置了。
```c++
current_offset = sc->chunk_offsets[i];
e->pos = current_offset;
current_offset += sample_size;
```

#### timestamp，当前帧的解码时间
timestamp 字段的值是通过 stts 表算出来的，stts 表的数据是压缩后的数据，记录了每一帧之间的增量时间，也就是解码时间差，如下：

所以，`mov_build_index()` 里面只需要将 `current_dts` 变量跟 `stts` 表的数据进行不断叠加，就能得到每一帧的解码时间。
```c++
e->timestamp = current_dts;
current_dts += sc->stts_data[stts_index].duration;
```

#### flag，帧标示
`flag` 字段有两个值，一是 `AVINDEX_KEYFRAME`，二是 `AVINDEX_DISCARD_FRAME`。

这两个值是可以共存的哈，`AVINDEX_KEYFRAME` 代表这一帧数据是关键帧，是通过 `stss`（关键帧表）来算出来的。

`AVINDEX_DISCARD_FRAME` 这个 flag 比较少用，如果被设置了 `AVINDEX_DISCARD_FRAME`，那会导致 `mov_read_packet()` 的时候丢弃这个 AVPacket，
```C++
e->flags = keyframe ? AVINDEX_KEYFRAME : 0;

for (i = index_entry_pos; i < st->nb_index_entries; i++) {
	if (prev_dts < st->index_entries[i].timestamp)
		break;
	st->index_entries[i].flags |= AVINDEX_DISCARD_FRAME;
}

if (sample->flags & AVINDEX_DISCARD_FRAME) {
	pkt->flags |= AV_PKT_FLAG_DISCARD;
}

```
#### size，帧数据的大小

size 字段是通过 stsz 表计算出来的，如下：

```c++
AVIndexEntry *e;
if (sample_size > 0x3FFFFFFF) {
	av_log(mov->fc, AV_LOG_ERROR, "Sample size %u is too large\n", sample_size);
	return;
}
e = &st->index_entries[st->nb_index_entries++];
e->pos = current_offset;
e->timestamp = current_dts;
e->size = sample_size;
e->min_distance = distance;
e->flags = keyframe ? AVINDEX_KEYFRAME : 0;
```

`flag` 与 `size` 的定义是有点奇怪的，如下：

```
int flags:2; 
int size:30;
```

他这种写法代表 flags 只占 2 位，size 只占 30 位，这样就只占 32 位，如果定义两个 int，内存就会翻一倍。


#### min_distance，当前帧与上一个关键帧的距离

`min_distance` 记录的是 当前帧与关键帧的**最近的**距离，实际上就是跟上一个关键帧的距离，他的算法是这样的，只要碰到关键帧，就把 `distance` 重置为 0，如果在遍历每一帧的时候，没碰到关键帧，`distance` 就会不断累计

```c++

for(i = 0; i < sc->chunk_count; i++){
	for(j = 0; j < sc->stsc_data[stsc_index].count; j++){
		······
		if (keyframe)
			distance = 0;
		······
		e->min_distance = distance;
	
		distance++;
	}
}
```





# 二，avformat_find_streaminfo，提取各个流的信息

![[Pasted image 20230814213144.jpg]]

第一个for循环
![[Pasted image 20230816005142.png]]

```c++
for (i = 0; i < ic->nb_streams; i++) {
	const AVCodec *codec;
	AVDictionary *thread_opt = NULL;
	st = ic->streams[i];
	avctx = st->internal->avctx;

	if (st->codecpar->codec_type == AVMEDIA_TYPE_VIDEO ||
		st->codecpar->codec_type == AVMEDIA_TYPE_SUBTITLE) {
/*            if (!st->time_base.num)
			st->time_base = */
		if (!avctx->time_base.num)
			avctx->time_base = st->time_base;
	}

	// only for the split stuff
	if (!st->parser && !(ic->flags & AVFMT_FLAG_NOPARSE) && st->request_probe <= 0) {

		st->parser = av_parser_init(st->codecpar->codec_id);

	ret = avcodec_parameters_to_context(avctx, st->codecpar);
	

	codec = find_probe_decoder(ic, st, st->codecpar->codec_id);



	/* Ensure that subtitle_header is properly set. */
	if (st->codecpar->codec_type == AVMEDIA_TYPE_SUBTITLE
		&& codec && !avctx->codec) {
		if (avcodec_open2(avctx, codec, options ? &options[i] : &thread_opt) < 0)
			av_log(ic, AV_LOG_WARNING,
				   "Failed to open codec in %s\n",__FUNCTION__);
	}
}
```

第二个for循环
```c++
for (i = 0; i < ic->nb_streams; i++) {
#if FF_API_R_FRAME_RATE
	ic->streams[i]->info->last_dts = AV_NOPTS_VALUE;
#endif
	ic->streams[i]->info->fps_first_dts = AV_NOPTS_VALUE;
	ic->streams[i]->info->fps_last_dts  = AV_NOPTS_VALUE;
}
```

第三个for循环
![[Pasted image 20230816005358.png]]

```c++
for (;;) {
	const AVPacket *pkt;
	int analyzed_all_streams;
	if (ff_check_interrupt(&ic->interrupt_callback)) {
		ret = AVERROR_EXIT;
		av_log(ic, AV_LOG_DEBUG, "interrupted\n");
		break;
	}

	/* check if one codec still needs to be handled */
	for (i = 0; i < ic->nb_streams; i++) {
		int fps_analyze_framecount = 20;
		int count;

		st = ic->streams[i];
		if (!has_codec_parameters(st, NULL))
			break;
		/* If the timebase is coarse (like the usual millisecond precision
		 * of mkv), we need to analyze more frames to reliably arrive at
		 * the correct fps. */
		if (av_q2d(st->time_base) > 0.0005)
			fps_analyze_framecount *= 2;
		if (!tb_unreliable(st->internal->avctx))
			fps_analyze_framecount = 0;
		if (ic->fps_probe_size >= 0)
			fps_analyze_framecount = ic->fps_probe_size;
		if (st->disposition & AV_DISPOSITION_ATTACHED_PIC)
			fps_analyze_framecount = 0;
		/* variable fps and no guess at the real fps */
		count = (ic->iformat->flags & AVFMT_NOTIMESTAMPS) ?
				   st->info->codec_info_duration_fields/2 :
				   st->info->duration_count;
		if (!(st->r_frame_rate.num && st->avg_frame_rate.num) &&
			st->codecpar->codec_type == AVMEDIA_TYPE_VIDEO) {
			if (count < fps_analyze_framecount)
				break;
		}
		// Look at the first 3 frames if there is evidence of frame delay
		// but the decoder delay is not set.
		if (st->info->frame_delay_evidence && count < 2 && st->internal->avctx->has_b_frames == 0)
			break;
		if (!st->internal->avctx->extradata &&
			(!st->internal->extract_extradata.inited ||
			 st->internal->extract_extradata.bsf) &&
			extract_extradata_check(st))
			break;
		if (st->first_dts == AV_NOPTS_VALUE &&
			!(ic->iformat->flags & AVFMT_NOTIMESTAMPS) &&
			st->codec_info_nb_frames < ((st->disposition & AV_DISPOSITION_ATTACHED_PIC) ? 1 : ic->max_ts_probe) &&
			(st->codecpar->codec_type == AVMEDIA_TYPE_VIDEO ||
			 st->codecpar->codec_type == AVMEDIA_TYPE_AUDIO))
			break;
	}
	analyzed_all_streams = 0;
	if (!missing_streams || !*missing_streams)
		if (i == ic->nb_streams) {
			analyzed_all_streams = 1;
			/* NOTE: If the format has no header, then we need to read some
			 * packets to get most of the streams, so we cannot stop here. */
			if (!(ic->ctx_flags & AVFMTCTX_NOHEADER)) {
				/* If we found the info for all the codecs, we can stop. */
				ret = count;
				av_log(ic, AV_LOG_DEBUG, "All info found\n");
				flush_codecs = 0;
				break;
			}
		}
	/* We did not get all the codec info, but we read too much data. */
	if (read_size >= probesize) {
		ret = count;
		av_log(ic, AV_LOG_DEBUG,
			   "Probe buffer size limit of %"PRId64" bytes reached\n", probesize);
		for (i = 0; i < ic->nb_streams; i++)
			if (!ic->streams[i]->r_frame_rate.num &&
				ic->streams[i]->info->duration_count <= 1 &&
				ic->streams[i]->codecpar->codec_type == AVMEDIA_TYPE_VIDEO &&
				strcmp(ic->iformat->name, "image2"))
				av_log(ic, AV_LOG_WARNING,
					   "Stream #%d: not enough frames to estimate rate; "
					   "consider increasing probesize\n", i);
		break;
	}
	/* NOTE: A new stream can be added there if no header in file
	 * (AVFMTCTX_NOHEADER). */
	ret = read_frame_internal(ic, &pkt1);
	if (ret == AVERROR(EAGAIN))
		continue;

	if (ret < 0) {
		/* EOF or error*/
		eof_reached = 1;
		break;
	}

	if (!(ic->flags & AVFMT_FLAG_NOBUFFER)) {
		ret = ff_packet_list_put(&ic->internal->packet_buffer,
								 &ic->internal->packet_buffer_end,
								 &pkt1, 0);
		if (ret < 0)
			goto unref_then_goto_end;

		pkt = &ic->internal->packet_buffer_end->pkt;
	} else {
		pkt = &pkt1;
	}

	st = ic->streams[pkt->stream_index];
	if (!(st->disposition & AV_DISPOSITION_ATTACHED_PIC))
		read_size += pkt->size;

	avctx = st->internal->avctx;
	if (!st->internal->avctx_inited) {
		ret = avcodec_parameters_to_context(avctx, st->codecpar);
		if (ret < 0)
			goto unref_then_goto_end;
		st->internal->avctx_inited = 1;
	}

	if (pkt->dts != AV_NOPTS_VALUE && st->codec_info_nb_frames > 1) {
		/* check for non-increasing dts */
		if (st->info->fps_last_dts != AV_NOPTS_VALUE &&
			st->info->fps_last_dts >= pkt->dts) {
			av_log(ic, AV_LOG_DEBUG,
				   "Non-increasing DTS in stream %d: packet %d with DTS "
				   "%"PRId64", packet %d with DTS %"PRId64"\n",
				   st->index, st->info->fps_last_dts_idx,
				   st->info->fps_last_dts, st->codec_info_nb_frames,
				   pkt->dts);
			st->info->fps_first_dts =
			st->info->fps_last_dts  = AV_NOPTS_VALUE;
		}
		/* Check for a discontinuity in dts. If the difference in dts
		 * is more than 1000 times the average packet duration in the
		 * sequence, we treat it as a discontinuity. */
		if (st->info->fps_last_dts != AV_NOPTS_VALUE &&
			st->info->fps_last_dts_idx > st->info->fps_first_dts_idx &&
			(pkt->dts - (uint64_t)st->info->fps_last_dts) / 1000 >
			(st->info->fps_last_dts     - (uint64_t)st->info->fps_first_dts) /
			(st->info->fps_last_dts_idx - st->info->fps_first_dts_idx)) {
			av_log(ic, AV_LOG_WARNING,
				   "DTS discontinuity in stream %d: packet %d with DTS "
				   "%"PRId64", packet %d with DTS %"PRId64"\n",
				   st->index, st->info->fps_last_dts_idx,
				   st->info->fps_last_dts, st->codec_info_nb_frames,
				   pkt->dts);
			st->info->fps_first_dts =
			st->info->fps_last_dts  = AV_NOPTS_VALUE;
		}

		/* update stored dts values */
		if (st->info->fps_first_dts == AV_NOPTS_VALUE) {
			st->info->fps_first_dts     = pkt->dts;
			st->info->fps_first_dts_idx = st->codec_info_nb_frames;
		}
		st->info->fps_last_dts     = pkt->dts;
		st->info->fps_last_dts_idx = st->codec_info_nb_frames;
	}
	if (st->codec_info_nb_frames>1) {
		int64_t t = 0;
		int64_t limit;

		if (st->time_base.den > 0)
			t = av_rescale_q(st->info->codec_info_duration, st->time_base, AV_TIME_BASE_Q);
		if (st->avg_frame_rate.num > 0)
			t = FFMAX(t, av_rescale_q(st->codec_info_nb_frames, av_inv_q(st->avg_frame_rate), AV_TIME_BASE_Q));

		if (   t == 0
			&& st->codec_info_nb_frames>30
			&& st->info->fps_first_dts != AV_NOPTS_VALUE
			&& st->info->fps_last_dts  != AV_NOPTS_VALUE)
			t = FFMAX(t, av_rescale_q(st->info->fps_last_dts - st->info->fps_first_dts, st->time_base, AV_TIME_BASE_Q));

		if (analyzed_all_streams)                                limit = max_analyze_duration;
		else if (avctx->codec_type == AVMEDIA_TYPE_SUBTITLE) limit = max_subtitle_analyze_duration;
		else                                                     limit = max_stream_analyze_duration;

		if (t >= limit) {
			av_log(ic, AV_LOG_VERBOSE, "max_analyze_duration %"PRId64" reached at %"PRId64" microseconds st:%d\n",
				   limit,
				   t, pkt->stream_index);
			if (ic->flags & AVFMT_FLAG_NOBUFFER)
				av_packet_unref(&pkt1);
			break;
		}
		if (pkt->duration) {
			if (avctx->codec_type == AVMEDIA_TYPE_SUBTITLE && pkt->pts != AV_NOPTS_VALUE && st->start_time != AV_NOPTS_VALUE && pkt->pts >= st->start_time) {
				st->info->codec_info_duration = FFMIN(pkt->pts - st->start_time, st->info->codec_info_duration + pkt->duration);
			} else
				st->info->codec_info_duration += pkt->duration;
			st->info->codec_info_duration_fields += st->parser && st->need_parsing && avctx->ticks_per_frame ==2 ? st->parser->repeat_pict + 1 : 2;
		}
	}
	if (st->codecpar->codec_type == AVMEDIA_TYPE_VIDEO) {
#if FF_API_R_FRAME_RATE
		ff_rfps_add_frame(ic, st, pkt->dts);
#endif
		if (pkt->dts != pkt->pts && pkt->dts != AV_NOPTS_VALUE && pkt->pts != AV_NOPTS_VALUE)
			st->info->frame_delay_evidence = 1;
	}
	if (!st->internal->avctx->extradata) {
		ret = extract_extradata(st, pkt);
		if (ret < 0)
			goto unref_then_goto_end;
	}

	/* If still no information, we try to open the codec and to
	 * decompress the frame. We try to avoid that in most cases as
	 * it takes longer and uses more memory. For MPEG-4, we need to
	 * decompress for QuickTime.
	 *
	 * If AV_CODEC_CAP_CHANNEL_CONF is set this will force decoding of at
	 * least one frame of codec data, this makes sure the codec initializes
	 * the channel configuration and does not only trust the values from
	 * the container. */
	try_decode_frame(ic, st, pkt,
					 (options && i < orig_nb_streams) ? &options[i] : NULL);

	if (ic->flags & AVFMT_FLAG_NOBUFFER)
		av_packet_unref(&pkt1);

	st->codec_info_nb_frames++;
	count++;
    }
```


第四个for循环
![[Pasted image 20230817102938.png]]


预测输入文件的时长

预测文件时长有 三种 策略：
**1，针对 mpeg 与 mpegts 格式，采用 `estimate_timings_from_pts()` 策略，他的算法是 用数据流 最后一个 AVPacket 的 PTS 减去 第一个 AVPacket 的 PTS，以此获取时长。
2，如果文件里其中一个流有准确的时长，就以此为标准 复制给其他流。
3，通过 码率 来预测，`estimate_timings_from_bit_rate()` 会获取文件的大小，然后除以 码率 得到时长。
```c++
if (probesize)
	estimate_timings(ic, old_offset);
```

```c++
if ((!strcmp(ic->iformat->name, "mpeg") ||
	 !strcmp(ic->iformat->name, "mpegts")) &&
	file_size && (ic->pb->seekable & AVIO_SEEKABLE_NORMAL)) {
	/* get accurate estimate from the PTSes */
	estimate_timings_from_pts(ic, old_offset);
	ic->duration_estimation_method = AVFMT_DURATION_FROM_PTS;
} else if (has_duration(ic)) {
	/* at least one component has timings - we use them for all
	 * the components */
	fill_all_stream_timings(ic);
	/* nut demuxer estimate the duration from PTS */
	if(!strcmp(ic->iformat->name, "nut"))
		ic->duration_estimation_method = AVFMT_DURATION_FROM_PTS;
	else
		ic->duration_estimation_method = AVFMT_DURATION_FROM_STREAM;
} else {
	/* less precise: use bitrate info */
	estimate_timings_from_bit_rate(ic);
	ic->duration_estimation_method = AVFMT_DURATION_FROM_BITRATE;
}
```

# 三，av_read_frame，读取 AVPacket 编码数据