# 阶段 2：应用层通信与 Web 服务基础

## 2.1 学习定位

HTTP 属于应用层协议，规定客户端和服务器如何表达、发送、理解业务消息。本阶段首次学习以就业和工程够用为目标：能读懂 HTTP 请求/响应，使用 Burp Suite 或 `curl` 观察接口，区分 DNS、TCP、TLS、HTTP 和业务层问题，并理解登录鉴权、HTTPS 和 WebSocket。

暂不深入 HTTP/2 帧结构、HTTP/3/QUIC、OAuth2/OIDC、复杂缓存和高并发连接池，遇到实际场景再专项学习。

## 2.2 HTTP 与网络分层

```text
应用层：HTTP、DNS、TLS、WebSocket
传输层：TCP、UDP、端口
网络层：IP、路由
链路层：Wi-Fi、以太网、MAC
```

典型 HTTPS 链路：

```text
HTTP 业务数据 -> TLS 加密 -> TCP 传输 -> IP 路由 -> Wi-Fi/以太网
```

HTTP 不负责寻找目标设备，也不负责底层可靠传输。

## 2.3 请求与响应

```text
客户端 -> HTTP 请求 -> 服务器
客户端 <- HTTP 响应 <- 服务器
```

普通 HTTP 通常是请求—响应模式；需要服务端主动推送时，可使用轮询、长轮询、SSE 或 WebSocket。

## 2.4 HTTP 请求结构

```text
请求行
请求头
空行
请求体（可选）
```

```http
POST /api/login HTTP/1.1
Host: game.example.com
Content-Type: application/json
Accept: application/json
User-Agent: GameClient/1.0
Content-Length: 52

{"username":"alice","password":"example-password"}
```

请求行包含请求方法、路径和 HTTP 版本。

### 常见方法

| 方法 | 常见用途 |
| --- | --- |
| GET | 获取资源 |
| POST | 提交数据、创建资源或触发状态变化 |
| PUT | 整体更新 |
| PATCH | 部分更新 |
| DELETE | 删除资源 |
| HEAD | 只获取响应头 |
| OPTIONS | 查询能力，常见于跨域预检 |

实际接口不一定严格遵循 REST。登录、使用道具、开始匹配都可能使用 POST。

### URL

```text
https://api.example.com:443/player/profile?id=1001#inventory
```

```text
协议 https | 主机 api.example.com | 端口 443
路径 /player/profile | 查询参数 id=1001 | 片段 inventory
```

`#inventory` 通常只由浏览器本地使用，不发送给服务器。

### 常见 Header

- `Host`：目标域名。同一 IP 可托管多个域名，服务器据此选择虚拟主机。
- `Content-Type`：请求体或响应体的实际/声明格式，如 `application/json`。
- `Accept`：客户端希望收到的格式。
- `Content-Length`：正文长度，帮助判断消息边界。
- `Authorization: Bearer <token>`：身份凭据。
- `User-Agent`：客户端类型和版本，可伪造，不能作为可靠身份认证。
- `Cookie`：客户端保存并发送的数据，如 `session_id=abc123`。

```text
Content-Type：我发送或收到的内容是什么格式
Accept：我希望收到什么格式
```

## 2.5 请求体

请求体常用于 POST、PUT、PATCH：

```json
{"username":"alice","password":"example-password"}
```

GET 常使用查询参数。POST 将数据放在 Body 中不代表自动加密；只有 HTTPS/TLS 才保护传输过程。服务端仍要校验字段格式、类型、权限和业务规则。

## 2.6 HTTP 响应结构

```text
状态行
响应头
空行
响应体（可选）
```

```http
HTTP/1.1 200 OK
Content-Type: application/json
Set-Cookie: session_id=abc123; HttpOnly; Secure

{"code":0,"message":"login success"}
```

响应体可以是 HTML、JSON、图片、音视频、文件或二进制数据。

## 2.7 状态码

```text
1xx 提示 | 2xx 成功 | 3xx 重定向 | 4xx 请求/权限问题 | 5xx 服务端问题
```

| 状态码 | 含义 |
| --- | --- |
| 200 | 请求成功 |
| 201 | 创建成功 |
| 204 | 成功但无正文 |
| 301/302 | 重定向 |
| 304 | 未修改，可用缓存 |
| 400 | 请求格式或参数错误 |
| 401 | 未登录、凭据缺失或无效 |
| 403 | 权限不足或策略拒绝 |
| 404 | 路径、资源或路由不存在 |
| 405 | 方法不允许 |
| 429 | 请求过于频繁 |
| 500 | 服务端内部错误 |
| 502 | 网关收到无效上游响应 |
| 503 | 服务暂时不可用 |
| 504 | 网关等待上游超时 |

```text
401：重点是“你是谁”
403：重点是“你有没有权限”
```

HTTP 状态码不等于业务结果：`HTTP 200 + code=10001` 可能表示余额不足、密码错误或其他业务失败。必须同时看响应体中的业务码和消息。

## 2.8 Cookie 与 Session

```text
客户端提交账号密码
 -> 服务器验证并创建 Session
 -> Set-Cookie 返回 session_id
 -> 客户端后续请求通过 Cookie 携带
 -> 服务器根据 session_id 找到会话状态
```

```text
Cookie：客户端保存并发送的数据载体
session_id：常见的会话索引
Session：服务器端保存的会话状态
```

Cookie 不等于用户身份本身，也可保存语言、主题和购物车标识。

Cookie 属性：

- `HttpOnly`：JavaScript 不能通过 `document.cookie` 直接读取，降低 XSS 窃取 Cookie 的风险，但不能阻止脚本代表用户发请求。
- `Secure`：只允许通过 HTTPS 发送，防止 Cookie 经普通 HTTP 传输；不是 HTTPS 的替代品。
- `SameSite`：限制跨站请求发送 Cookie，可降低 CSRF 风险。
- `Path`：限制生效路径。

## 2.9 Token 与 JWT

Token 通常由服务器生成，常见发送方式：

```http
Authorization: Bearer <token>
```

客户端可以伪造字符串，但正常情况下不能生成通过服务器签名、有效期和权限检查的有效 Token。

JWT 通常由三部分组成：

```text
Header.Payload.Signature
```

Payload 可能包含用户 ID、角色和过期时间。JWT 默认是 Base64URL 编码加签名，不是加密；Payload 通常可以被解码查看。

```text
签名：验证来源和防篡改
加密：防止别人读取内容
```

常见的 Access Token 较短期，Refresh Token 用于换取新的 Access Token。不要把密码、银行卡信息、私密聊天内容或密钥放进普通 JWT。

## 2.10 CSRF

CSRF（跨站请求伪造）利用浏览器自动携带 Cookie，诱导已登录用户向目标网站发送危险请求。

```text
已登录目标网站 -> 打开恶意网站 -> 恶意页面诱导请求
-> 浏览器自动携带目标 Cookie -> 目标误以为是用户操作
```

```text
CSRF：别人借用你的身份发请求
XSS：恶意脚本在目标页面上下文执行
```

常见防护：CSRF Token、`SameSite` Cookie、校验 `Origin/Referer`、使用显式设置的 Authorization Header。原生 UE 客户端通常不直接受到浏览器 CSRF 规则影响，但 Web 登录、商城和后台仍可能存在。

## 2.11 HTTPS、TLS 与证书

```text
HTTPS = HTTP + TLS
```

TLS 提供：

- 保密性：中间人不易直接读取内容。
- 完整性：篡改能够被检测。
- 身份认证：客户端验证服务器证书。

简化过程：

```text
连接服务器 443
 -> 协商 TLS
 -> 服务器发送证书
 -> 客户端验证域名、有效期和证书链
 -> 建立会话密钥
 -> 加密 HTTP 数据
```

HTTPS 保护传输，不保证服务端业务逻辑、密码存储、权限校验或客户端本身安全。服务器仍需做格式、身份、权限和业务校验。

### 80 与 443

```text
80：HTTP 默认端口
443：HTTPS 默认端口
```

它们是约定俗成的默认端口，不是强制限制。HTTP 可运行在 8080，HTTPS 可运行在 8443。80 常用于重定向到 HTTPS、兼容旧链接和证书验证。

## 2.12 轮询、长轮询与 WebSocket

普通轮询是客户端定时询问：无消息时也会产生请求，实时性受间隔影响。

长轮询是服务器暂不立即响应，等有消息或超时后再返回；客户端收到后重新请求。

WebSocket 通常通过 HTTP Upgrade 建立：

```http
GET /chat HTTP/1.1
Upgrade: websocket
Connection: Upgrade
```

服务器返回 `101 Switching Protocols` 后，连接变成双方都可主动发送消息的长连接。WebSocket 通常建立在 TCP 之上，具备可靠、有序、字节流特性，但不等于完整的高频游戏网络方案。

```text
HTTP：请求 -> 响应
WebSocket：建立长连接后，双方均可主动通信
```

## 2.13 HTTP 与游戏客户端

```text
HTTPS：登录、公告、配置、背包、商城、排行榜、支付回调
WebSocket/其他长连接：聊天、大厅事件、房间状态
专用游戏网络：实时对局、移动、技能和状态同步
```

## 2.14 超时、重试与幂等性

```text
请求超时 != 请求一定没有到达服务器
```

购买请求可能已在服务端执行，但响应返回途中丢失；客户端盲目重试可能造成重复购买。

重试前要判断：请求是否安全、服务端是否可能已执行、是否有幂等键/请求 ID、是否需要指数退避。查询类 GET 通常较适合重试；扣款、领奖、创建订单等改变状态的 POST 不能盲目重试。

## 2.15 CORS 基础

CORS（跨源资源共享）是浏览器的安全机制。网页向不同协议、域名或端口请求时，浏览器可能限制页面读取响应。服务器可通过：

```http
Access-Control-Allow-Origin: https://game.example.com
```

声明允许来源；复杂请求可能先发送 `OPTIONS` 预检。Burp 能看到请求，不代表浏览器页面一定允许 JavaScript 读取响应。原生 UE 客户端通常不直接受 CORS 限制。

## 2.16 Range 与简单缓存

视频等大文件常使用范围请求：

```http
Range: bytes=557056-580510
If-Range: "etag-value"
```

服务器可能返回：

```http
HTTP/2 206 Partial Content
Content-Range: bytes 557056-580510/10485760
Content-Length: 23455
```

这表示只返回文件的一段。`ETag` 是资源版本标识，`If-Range`/`If-None-Match` 用于缓存验证，`Cache-Control` 表示缓存策略，`Accept-Ranges: bytes` 表示支持按字节请求。

Burp 中同一路径出现多条 Range 请求是正常的；完整视频需要把多条请求和响应放在一起分析。

## 2.17 Burp Suite 与 curl

Burp Suite 适合 HTTP/HTTPS 应用层：方法、URL、Header、Cookie、Token、JSON、文件上传、响应和接口重放。HTTPS 抓包通常需要浏览器信任 Burp CA 证书。

```bash
curl.exe -I https://example.com
curl.exe -I -L https://example.com
curl.exe -v --connect-timeout 10 https://example.com
```

`-I` 只请求响应头，`-L` 跟随重定向，`-v` 查看详细连接过程：

```text
Host was resolved -> DNS 完成
Connected          -> TCP 建立
SSL connection     -> TLS 建立
>                  -> 客户端请求
<                  -> 服务器响应
```

## 2.18 HTTP/2 请求看起来不完整的原因

Burp 中看到 `HTTP/2` 的视频请求时，可能只有一段路径和一个 `Range`。原因通常是：HTTP/2 底层使用二进制帧；视频采用分段下载；浏览器使用缓存和 `If-Range`。这通常是一个完整的分段请求，不是损坏的请求。

## 2.19 阶段 2 收官标准

- 能读懂请求行、Header、Body、状态行和响应体。
- 能区分 `Content-Type` 与 `Accept`、HTTP 状态码与业务状态码。
- 能说明 Cookie、Session、Token、JWT 的基本关系。
- 能解释 HTTPS/TLS 的保护范围与局限。
- 能区分 HTTP、轮询、长轮询和 WebSocket。
- 能用 Burp 和 `curl -v` 观察接口并判断大致卡在 DNS、TCP、TLS 还是 HTTP。
- 能理解超时、重试、幂等性、CORS 和 Range 请求的基础含义。

更深的缓存、安全、JWT、TLS、HTTP/2 和 HTTP/3 内容，等实际项目出现需求时再专项学习。
