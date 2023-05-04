---
doc_type: hypothesis-highlights
url: 'https://blog.csdn.net/BinBinCome/article/details/130009978'
---

# C++轻量级Web服务器TinyWebServer源码分析之http篇_BingeBlog的博客-CSDN博客

## Metadata
- Author: [blog.csdn.net]()
- Title: C++轻量级Web服务器TinyWebServer源码分析之http篇_BingeBlog的博客-CSDN博客
- Reference: https://blog.csdn.net/BinBinCome/article/details/130009978
- Category: #article

## Page Notes
## Highlights
- 主状态机 三种状态，标识解析位置。 CHECK_STATE_REQUESTLINE，解析请求行 CHECK_STATE_HEADER，解析请求头 CHECK_STATE_CONTENT，解析消息体，仅用于解析POST请求 从状态机 三种状态，标识解析一行的读取状态。 LINE_OK，完整读取一行 LINE_BAD，报文语法有误 LINE_OPEN，读取的行不完整 — [Updated on 2023-04-14 01:14:45](https://hyp.is/sAYPUtoeEe2SzAc0bcFV8Q/blog.csdn.net/BinBinCome/article/details/130009978) — Group: #Public

- 对于主从状态机解析的结果定义为HTTP_CODE，分别为 NO_REQUEST//请求不完整，需要继续读取请求报文数据 GET_REQUEST//获得了完整的HTTP请求 BAD_REQUEST //HTTP请求报文有语法错误 INTERNAL_ERROR//服务器内部错误，该结果在主状态机逻辑switch的default下，一般不会触发 — [Updated on 2023-04-14 01:15:15](https://hyp.is/wgNEkNoeEe2Szes9qYwWLQ/blog.csdn.net/BinBinCome/article/details/130009978) — Group: #Public

- 解析报文整体流程 process_read通过while循环，将主从状态机进行封装，对报文的每一行进行循环处理。 在HTTP报文中，每一行的数据由\r\n作为结束字符，空行则是仅仅是字符\r\n。因此，可以通过查找\r\n将报文拆解成单独的行进行解析。 判断条件 主状态机转移到CHECK_STATE_CONTENT，该条件涉及解析消息体 从状态机转移到LINE_OK，该条件涉及解析请求行和请求头部 两者为或关系，当条件为真则继续循环，否则退出 — [Updated on 2023-04-14 01:17:35](https://hyp.is/FaGxStofEe2MYSNEZ6I_2w/blog.csdn.net/BinBinCome/article/details/130009978) — Group: #Public

- 循环体 从状态机读取数据 调用get_line函数，通过m_start_line将从状态机读取数据间接赋给text 主状态机解析text — [Updated on 2023-04-14 01:17:38](https://hyp.is/F0hd8NofEe21C7eJ4jJFPg/blog.csdn.net/BinBinCome/article/details/130009978) — Group: #Public

- 从状态机逻辑 从状态机负责读取buffer中的数据，将每行数据末尾的\r\n置为\0\0，并更新从状态机在buffer中读取的位置m_checked_idx，以此来驱动主状态机解析。 — [Updated on 2023-04-14 01:17:45](https://hyp.is/GyN2stofEe2uE4MpoZnTLQ/blog.csdn.net/BinBinCome/article/details/130009978) — Group: #Public







