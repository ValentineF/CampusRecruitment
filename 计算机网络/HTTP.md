# 0.HTTP方法
| 方法 | 作用 | 详细 |
| --- | --- | --- |
| GET |获取资源| 安全性差，在URL中明文传输 |
| POST |传输实体主体| 告诉服务端信息，用于提交或修改|
| HEAD |获取报文首部|确认URL的有效性及资源更新的时间|
| PUT |上传文件|无验证机制，不安全|
| DELETE | 删除文件 |无验证机制，不安全|
| OPTIONS | 查询支持的方法 |会返回 Allow: GET, POST, HEAD, OPTIONS 这样的内容。|
| TRACE | 追踪路径 | 容易受到XST攻击|
| CONNECT| 要求用隧道协议连接代理 |使用SSL/TLS加密后进入隧道传输|
# 1.Get/Post方法的区别
    0.Get用于获取数据，Post用于提交数据
    1.GET产生一个TCP数据包;POST产生两个TCP数据包（所以，Get更快）
        A.GET，浏览器会把header和data一并发送出去，服务器响应200(返回数据);
        B.POST，浏览器先发送header，服务器响应100 continue，浏览器再发送data，服务器响应200 ok(返回数据)。
    2.Get请求会被浏览器主动缓存,Post不会
    3.Get只能进行Url编码，Post可以进行多种编码
    4.Get请求的参数有长度限制（主要是URL限制），Post没有
    5.Get的参数暴露在Url中不安全，其实Post也不安全（可以F12或者抓包）
    6.GET参数通过Url传递，POST放在Request body中。
# 2.HTTP状态码
服务端送回的响应报文里包含了状态码和原因短语
| 状态码 | 类别 | 原因短语 |
| --- | --- | --- |
| 1XX | Informational（信息性状态码） | 接收的请求正在处理 |
| 2XX | Success（成功状态码） | 请求正常处理完毕 |
| 3XX | Redirection（重定向状态码） | 需要进行附加操作以完成请求 |
| 4XX | Client Error（客户端错误状态码） | 服务器无法处理请求 |
| 5XX | Server Error（服务器错误状态码） | 服务器处理请求出错 |

详细状态码
| 200 OK | 204 No Content | 206 Partial Content |
| --- | --- | --- |
| 301 Moved Permanently | 302 Found |
| 303 See Other | 304 Not Modified | 307 Temporary Redirect
| 400 Bad Request | 401 Unauthorized| 403 Forbidden | 404 Not Found |
| 500 Internal Server Error | 503 Service Unavilable |
# 3.请求报文和响应报文
请求报文   
![请求报文](https://github.com/ValentineF/CampusRecruitment/blob/master/Picture/%E8%AF%B7%E6%B1%82%E6%8A%A5%E6%96%87.jpg?raw=true)   
响应报文   
![响应报文](https://github.com/ValentineF/CampusRecruitment/blob/master/Picture/%E5%93%8D%E5%BA%94%E6%8A%A5%E6%96%87.jpg?raw=true)
# 4.HTTP首部（头）
分四种:通用首部字段、请求首部字段、响应首部字段、实体首部字段。
以通用首部字段为例
## 通用首部字段
| 首部字段名 | 说明 |
| -- | -- |
| Cache-Control | 控制缓存的行为 |
| Connection | 逐跳首部、 连接的管理 |
| Date | 创建报文的日期时间 |
| Pragma | 报文指令 |
| Trailer | 报文末端的首部一览 |
| Transfer-Encoding | 指定报文主体的传输编码方式 |
| Upgrade | 升级为其他协议 |
| Via | 代理服务器的相关信息 |
| Warning | 错误通知 |
## 通用首部字段之Cache-Control
用于指定所有缓存机制在整个请求/响应链中必须服从的指令。   
Expires 字段可以用于告知缓存服务器该资源什么时候会过期。（优先级低）
## Cache-Control字段值
| 字段值 | 说明 |
| --- | --- |
| public | 所有内容都将被缓存(客户端和代理服务器都可缓存) |
| private |	内容只缓存到私有缓存中(仅客户端可以缓存，代理服务器不可缓存) |
| no-cache | 必须先确认返回的响应是否被更改，才能使用该响应 |
| no-store	| 所有内容都不会被缓存到缓存或 Internet 临时文件中 |

# 5.Cookie/Session
    0.由于Http是无状态的，所以引进了Cookie来保存信息
    1.Cookie（客户端）：
        服务器发送的响应报文包含 Set-Cookie 字段，客户端得到响应报文后把 Cookie 内容保存到浏览器中。
        下次再发送请求时，从浏览器中读出 Cookie 值，在请求报文中包含 Cookie 字段，这样服务器就知道客户端的状态信息了。
        Cookie 状态信息保存在客户端浏览器中，而不是服务器上。
    2.Session（服务端）:
        Session 是服务器用来跟踪用户的一种手段，每个 Session 都有一个唯一标识：Session ID。
        当服务器创建了一个Session时，给客户端发送的响应报文就包含了 Set-Cookie字段，其中有一个名为sid的键值对，这个键值对就是Session ID。
        客户端收到后就把 Cookie 保存在浏览器中，并且之后发送的请求报文都包含 Session ID。
        HTTP 就是通过 Session 和 Cookie 这两种方式一起合作来实现跟踪用户状态的，Session 用于服务器端，Cookie 用于客户端。
# 6.HTTPS
    1.HTPP的安全问题：
        A.通信使用明文，内容可能会被窃听；
        B.不验证通信方的身份，因此有可能遭遇伪装；
        C.无法证明报文的完整性，所以有可能已遭篡改。
    2.HTTPS: 
        并不是新协议，而是 HTTP 先和 SSL通信，再由 SSL 和 TCP 通信。
        提供了加密、认证和完整性保护。
    3.加密机制：
        采用混合加密：使用【公开密钥加密】用于传输对称密钥，之后使用【对称密钥加密】进行通信。
# 7.持久连接
    0.HTTP1.0默认持久连接（即只需建立一次TCP连接就能进行多次HTTP通信）
    1.Connection 首部字段进行管理:[Close]即为断开
    2.管线化方式:同时发送多个请求和响应，不用发一回一。
# 8.其他常用概念
    0.编码：对实体进行压缩。常用的编码有：gzip、compress、deflate、identity，其中 identity 表示不执行压缩的编码格式。
    1.分块传输:数据分割成多块，让浏览器逐步显示页面
    2.范围请求
        A.如果网络出现中断，服务器只发送了一部分数据，范围请求使得客户端能够只请求未发送的那部分数据，从而避免服务器端重新发送所有数据。
        B.请求报文添加：Range字段，指定请求的范围；若成功,则返回206
    3.内容协商
        A.通过内容协商返回最合适的内容，例如根据浏览器的默认语言选择返回中文界面还是英文界面。
        B.涉及以下首部字段：Accept、Accept-Charset、Accept-Encoding、Accept-Language、Content-Language。
    4.通信数据转发
        A.代理：代理服务器接受客户端的请求，并且转发给其它服务器。
            作用:缓存、网络访问控制以及记录访问日志。 
        B.网关：将 HTTP 转化为其它协议进行通信，从而请求其它非 HTTP 服务器的服务
        C.隧道：使用 SSL 等加密手段，为客户端和服务器之间建立一条安全的通信线路。
    5.缓存：有两种缓存方法：让【代理服务器】进行缓存和让【客户端浏览器】进行缓存。
资料来源于《图解HTTP》和[CyC2018的笔记](https://github.com/CyC2018/Interview-Notebook/blob/master/notes/HTTP.md#%E5%85%B7%E4%BD%93%E5%BA%94%E7%94%A8)
