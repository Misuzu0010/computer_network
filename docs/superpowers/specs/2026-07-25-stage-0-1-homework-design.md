# 阶段 0 与阶段 1 作业设计

## 目标

检验阶段 0 的概念理解、阶段 1 的命令输出解读、本机观察和分层排错能力，不以考研式定义背诵为目标。

## 作业形式

采用引导式诊断作业，总分 100 分：

| 模块 | 内容 | 分值 |
| --- | --- | ---: |
| A | 核心概念辨析与短答 | 30 |
| B | 命令输出解读 | 30 |
| C | 本机命令实操与结果说明 | 25 |
| D | 网络故障场景的分层排查 | 15 |

## 范围

- A：IP、端口、进程、Socket、客户端/服务器、网关、分层、延迟、带宽、抖动、丢包。
- B：脱敏的 `ipconfig`、`ping`、`tracert`、`nslookup`、`curl` 输出。
- C：运行上述命令并提交必要的脱敏观察结果。
- D：域名解析失败、连接超时、连接被拒绝和 HTTP 状态码异常场景。

## 实操命令

```bash
ipconfig.exe
ping.exe -n 4 www.baidu.com
tracert.exe www.baidu.com
nslookup.exe -type=A www.baidu.com
curl.exe -I https://www.baidu.com
```

不提交公网 IP、MAC、Cookie、Token、Authorization、账号或完整本地配置。

## 评分原则

- 概念题重视因果解释。
- 输出题重视证据边界：Ping 成功不等于 HTTPS 正常；中间跳超时但最终抵达不等于网络中断。
- 实操题不因不同运营商、DNS、CDN、IP 或延迟数值而扣分。
- 场景题重视分层排查顺序，不要求猜中唯一原因。

## 本次不考查

- UE Replication、RPC 和多人 Gameplay 网络。
- TCP 拥塞控制公式、复杂子网计算和协议字段背诵。
- Socket 编程。
- Burp Suite 或 Wireshark 的实际抓包操作。
