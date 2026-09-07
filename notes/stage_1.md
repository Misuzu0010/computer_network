# 阶段 1：网络观察与基础排错

## 学习目标

把阶段 0 的概念对应到真实网络环境，能够查看本机配置、验证解析和连通性、观察路由、验证 HTTP 服务，并按层次定位问题。

## 排错顺序

```text
本机网络配置 -> DNS -> IP 与路由 -> TCP/UDP 端口 -> HTTP/HTTPS -> 业务逻辑
```

不要把所有问题都归因于代码或服务器。

## 常用命令

在 Git Bash 中调用 Windows 工具时可加 `.exe`：

```bash
ipconfig.exe
ipconfig.exe /all
ping.exe -n 4 www.baidu.com
tracert.exe www.baidu.com
nslookup.exe -type=A www.baidu.com
curl.exe -I https://www.baidu.com
curl.exe -v https://www.baidu.com
```

### `ipconfig`

重点观察当前联网适配器的 IPv4 地址、子网掩码、默认网关和 DNS。`169.254.x.x` 通常表示 DHCP 未成功分配正常地址。VPN、虚拟机和多个网卡可能改变路由与 DNS，不能随便选择适配器。

### `ping`

Ping 使用 ICMP，主要观察解析结果、目标是否回应和 RTT。Ping 成功不证明 TCP 端口或 HTTP 正常；Ping 失败也不证明网站一定不可访问，因为目标可能屏蔽 ICMP。

### `tracert`

通过逐步增加 TTL 观察大致路由跳数。第一跳通常是默认网关。某一跳显示 `*` 只说明该节点没有回应探测、限速或被过滤；若后续仍能到达目标，不应判断为网络在该处中断。

### `nslookup`

专门检查 DNS，将域名解析为 IP。一个域名对应多个 IP 可能是负载均衡、CDN、高可用或同时支持 IPv4/IPv6。`A` 是 IPv4，`AAAA` 是 IPv6，`CNAME` 是别名。

### `curl`

`curl -I` 只请求响应头，比 Ping 更接近真实 Web 访问。成功通常意味着已经完成 DNS、网络连接、TCP、TLS 和 HTTP 请求。常见结果：

| 结果 | 初步含义 |
| --- | --- |
| `200` | 请求成功 |
| `301/302` | 重定向 |
| `401` | 未登录或凭据无效 |
| `403` | 权限不足或被策略拒绝 |
| `404` | 路径或资源不存在 |
| `500` | 服务端内部错误 |
| `Could not resolve host` | DNS 失败 |
| `Connection refused` | 目标端口未监听或主动拒绝 |
| `Connection timed out` | 路由、端口、防火墙或服务可能超时 |

不要公开粘贴真实 Cookie、Token、Authorization 或账号信息。

## Burp Suite 与 Wireshark

```text
Burp Suite：HTTP/HTTPS 业务层，查看请求、响应、Header、Cookie、Token、JSON、重放接口
Wireshark：链路层到传输层，查看 DNS、IP、TCP 握手、重传、UDP、端口和网络异常
```

Burp Suite 适合调试 Web API 和登录鉴权；Wireshark 适合分析 TCP/UDP、DNS、丢包和未来的实时游戏网络。二者不是替代关系。

## 已完成的实际案例

执行百度的 `tracert` 后，最终在第 15 跳抵达目标，延迟约 9～10ms。第 1 跳 `10.154.0.1` 是默认网关；第 2、3 跳是内部网络；第 4 跳出现公网地址；中间多跳 `*` 只是设备不回应探测，因后续仍到达目标，不能判断为断网。

## 自测

1. 为什么 Ping 成功不能证明 HTTPS 可用？
2. `nslookup` 成功但 `curl` 超时，问题一定是 DNS 吗？
3. `tracert` 某跳为 `*` 但最终抵达，应如何理解？
4. `Connection refused` 与 `Connection timed out` 的排查方向有什么区别？
5. 调试 HTTP 登录请求应优先使用 Burp Suite 还是 Wireshark？
