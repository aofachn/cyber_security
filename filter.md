# Wireshark 与 tcpdump 过滤器完整指南

> **核心区别**：Wireshark 使用**显示过滤器**（Display Filter），在抓包后筛选已捕获的数据包，语法如 `tcp.port == 80`；tcpdump 使用 **BPF 捕获过滤器**（Capture Filter），在抓包前于内核层筛选，不符合条件的包根本不会被捕获，语法如 `tcp port 80`。两者语法体系完全不同，不可混用。

## 一、Wireshark 显示过滤器

### 1.1 基本语法规则

显示过滤器由**协议名**、**字段名**、**运算符**和**值**构成。最简形式是直接写协议名（如 `http`），更常用的是“字段 + 运算符 + 值”的组合。

**常用运算符速查**：

| 运算符 | 含义 | 示例 |
|:---|:---|:---|
| `==` | 等于 | `ip.addr == 192.168.1.1` |
| `!=` | 不等于 | `ip.addr != 10.0.0.1` |
| `>` `<` `>=` `<=` | 比较 | `tcp.len > 1000` |
| `contains` | 包含字符串 | `http.host contains "example"` |
| `matches` | 正则匹配 | `http.request.uri matches "gl=se$"` |
| `in` | 属于集合 | `tcp.port in {80, 443, 8080}` |
| `and` / `or` / `not` | 逻辑组合 | `tcp and not port 22` |

`in` 运算符支持范围写法：`tcp.port in {443, 4430..4434}` 表示端口 443 或 4430 到 4434 之间的任意端口。集合也可用于字符串：`http.request.method in {"GET", "POST"}`。

### 1.2 按协议筛选

    tcp
    udp
    http
    dns
    icmp
    arp
    tls
    http2

### 1.3 按 IP 地址筛选

    ip.addr == 192.168.1.100                 # 任意方向匹配该 IP（源或目的）
    ip.src == 10.0.0.5                       # 仅源 IP
    ip.dst == 172.16.0.1                     # 仅目的 IP
    ip.addr != 192.168.1.100                 # 排除该 IP
    ip.addr in {192.168.1.10, 192.168.1.20}  # 匹配两个 IP 之一
    ip.addr in {10.0.0.5 .. 10.0.0.9}        # IP 范围

注意：`ip.addr == X` 等价于 `ip.src == X or ip.dst == X`。

### 1.4 按端口筛选

    tcp.port == 80                           # TCP 任意方向 80 端口
    tcp.srcport == 54321                     # TCP 源端口
    tcp.dstport == 443                       # TCP 目的端口
    udp.port == 53                           # UDP 端口（DNS）
    tcp.port in {80, 443, 8080}              # 多个端口任选
    tcp.port in {443, 4430..4434}            # 端口 + 范围

### 1.5 HTTP 相关

    http.request.method == "GET"
    http.request.method in {"GET", "POST"}
    http.host contains "example"
    http.request.uri contains "/api/login"
    http.request.uri matches "gl=se$"        # 正则：URI 以 gl=se 结尾
    http.response.code >= 400                # 4xx/5xx 错误
    http.response.code in {404, 500, 502, 503}
    http.user_agent contains "curl"
    http.cookie contains "PHPSESSION"

### 1.6 TCP 标志位与异常

    tcp.flags.syn == 1                       # SYN 包
    tcp.flags.syn == 1 and tcp.flags.ack == 0    # 仅 SYN（非 SYN-ACK）
    tcp.flags.reset == 1                     # RST 包
    tcp.analysis.retransmission              # TCP 重传
    tcp.analysis.duplicate_ack               # 重复 ACK
    tcp.len > 0                              # 携带数据的 TCP 包
    tcp.stream == 5                          # 特定 TCP 流编号
    tcp.window_size == 0 && tcp.flags.reset != 1   # 窗口为零（接收方要求停止发送）

### 1.7 DNS 相关

    dns
    dns.qry.name contains "example.com"
    dns.flags.response == 0                  # DNS 查询
    dns.flags.response == 1                  # DNS 响应
    dns.time > 0.5                           # 响应时间超过 500ms

### 1.8 逻辑组合与排除

    tcp.port == 80 and ip.src == 10.0.0.5
    udp.port == 53 or tcp.port == 53
    not arp
    not (ip.addr == 192.168.1.1 and tcp.port == 22)
    http and not (http.response.code == 200)
    ip.src==192.168.0.0/16 and ip.dst==192.168.0.0/16   # 仅局域网内部流量

### 1.9 帧级与内容筛选

    frame.len > 1000                         # 帧长度大于 1000 字节
    frame.number >= 100 and frame.number <= 200
    frame.time >= "2026-10-24 10:00:00"
    frame contains "password"                # 帧内容包含指定字符串
    udp contains 81:60:03                    # UDP 头或载荷包含 3 字节序列
    udp[8:3] == 81:60:03                     # UDP 载荷起始 3 字节匹配（跳过 8 字节头）
    eth.addr[0:3] == 00:06:5B                # MAC 地址厂商 OUI 匹配

## 二、tcpdump 捕获过滤器（BPF）

### 2.1 基本语法结构

tcpdump 的过滤表达式由**类型限定符**（`host`、`net`、`port`）、**方向限定符**（`src`、`dst`）和**协议限定符**（`tcp`、`udp`、`icmp`、`arp`、`ip`）组合而成，用 `and` / `or` / `not` 连接。

基本命令结构：`tcpdump -i <接口> <过滤表达式>`

### 2.2 按主机与网段筛选

    tcpdump -i eth0 host 192.168.1.100               # 任意方向
    tcpdump -i eth0 src 192.168.1.100                # 仅源
    tcpdump -i eth0 dst 192.168.1.100                # 仅目的
    tcpdump -i eth0 not host 192.168.1.100           # 排除
    tcpdump -i eth0 net 192.168.1.0/24               # 整个网段
    tcpdump -i eth0 src net 10.0.0.0/8 and dst net 192.168.0.0/16   # 网段之间

### 2.3 按端口筛选

    tcpdump -i eth0 port 80                          # 80 端口（TCP/UDP）
    tcpdump -i eth0 tcp port 80                      # 仅 TCP 80
    tcpdump -i eth0 src port 22                      # 源端口 22
    tcpdump -i eth0 dst port 443                     # 目的端口 443
    tcpdump -i eth0 portrange 8000-9000              # 端口范围
    tcpdump -i eth0 port 80 or port 443              # 多端口
    tcpdump -i eth0 not port 22                      # 排除 SSH

### 2.4 按协议筛选

    tcpdump -i eth0 tcp
    tcpdump -i eth0 udp
    tcpdump -i eth0 icmp
    tcpdump -i eth0 arp
    tcpdump -i eth0 ip                               # 仅 IPv4
    tcpdump -i eth0 not arp

### 2.5 逻辑组合

    tcpdump -i eth0 host 192.168.1.100 and port 80
    tcpdump -i eth0 'tcp port 80 or tcp port 443'
    tcpdump -i eth0 'src 192.168.1.100 and dst port 3306'
    tcpdump -i eth0 'tcp and not port 22'
    tcpdump -i eth0 'host 10.0.0.1 and (port 80 or port 443)'
    tcpdump -i eth0 'not src 127.0.0.1'

### 2.6 TCP 标志位过滤（字节偏移）

BPF 允许直接访问 TCP 头字节。`tcp[13]` 是 TCP 标志位字节，`tcp[tcpflags]` 是等价写法。

    # SYN 包（连接建立请求）
    tcpdump -i eth0 'tcp[tcpflags] & tcp-syn != 0'

    # 仅 SYN（不含 ACK），即连接发起包
    tcpdump -i eth0 'tcp[tcpflags] & (tcp-syn|tcp-ack) == tcp-syn'

    # SYN-ACK 包
    tcpdump -i eth0 'tcp[tcpflags] & (tcp-syn|tcp-ack) == (tcp-syn|tcp-ack)'

    # RST 包
    tcpdump -i eth0 'tcp[tcpflags] & tcp-rst != 0'

    # FIN 或 RST
    tcpdump -i eth0 'tcp[tcpflags] & (tcp-fin|tcp-rst) != 0'

    # 等价写法：tcp[13] & 0x02 != 0（SYN）

### 2.7 HTTP 请求内容过滤

利用 TCP 载荷偏移定位 HTTP 方法起始位置。`0x47455420` 是 “GET ” 的十六进制，`0x504f5354` 是 “POST”。

    # 捕获 HTTP GET 请求
    tcpdump -i eth0 -A -s 0 'tcp port 80 and (tcp[((tcp[12:1] & 0xf0) >> 2):4] = 0x47455420)'

    # 捕获 HTTP POST
    tcpdump -i eth0 -A -s 0 'tcp port 80 and (tcp[((tcp[12:1] & 0xf0) >> 2):4] = 0x504f5354)'

### 2.8 DNS、ICMP 与其他

    tcpdump -i eth0 udp port 53                      # DNS
    tcpdump -i eth0 icmp                             # Ping/ICMP
    tcpdump -i eth0 'icmp[icmptype] = icmp-echo'     # 仅 Echo 请求（ping 请求）
    tcpdump -i eth0 'greater 1000'                   # 包长度大于 1000 字节
    tcpdump -i eth0 'less 100'                       # 包长度小于 100 字节

### 2.9 排除常见干扰

    # 只看非常见端口（排除 SSH/HTTP/HTTPS 干扰）
    tcpdump -i eth0 'not port 22 and not port 80 and not port 443'

    # 排除本地回环
    tcpdump -i eth0 'tcp and not host 127.0.0.1'

## 三、常用选项对照

| 选项 | tcpdump 作用 | 对应 Wireshark 场景 |
|:---|:---|:---|
| `-i any` / `-i eth0` | 指定网卡 | 抓包前选择接口 |
| `-nn` | 不解析主机名和端口名 | 显示过滤器用数字 IP/端口 |
| `-c N` | 抓 N 个包后停止 | 手动停止抓包 |
| `-w file.pcap` | 保存到文件 | 保存捕获文件 |
| `-r file.pcap` | 读取文件 | 打开捕获文件 |
| `-A` | 以 ASCII 显示载荷 | 追踪 TCP 流（Follow Stream） |
| `-X` | 十六进制 + ASCII 显示 | 查看原始字节 |

## 四、语法对照速查表

| 需求 | Wireshark 显示过滤器 | tcpdump 捕获过滤器 |
|:---|:---|:---|
| 指定 IP | `ip.addr == 192.168.1.1` | `host 192.168.1.1` |
| 指定源 IP | `ip.src == 10.0.0.1` | `src 10.0.0.1` |
| 指定目的 IP | `ip.dst == 10.0.0.1` | `dst 10.0.0.1` |
| 指定端口 | `tcp.port == 80` | `tcp port 80` |
| 排除端口 | `tcp.port != 22` | `not port 22` |
| 多端口 | `tcp.port in {80,443}` | `port 80 or port 443` |
| 端口范围 | `tcp.port in {443,4430..4434}` | `portrange 4430-4434` |
| 网段 | `ip.addr == 192.168.1.0/24` | `net 192.168.1.0/24` |
| 协议 | `icmp` | `icmp` |
| SYN 包 | `tcp.flags.syn == 1` | `tcp[tcpflags] & tcp-syn != 0` |
| 逻辑与 | `and` | `and` |
| 逻辑或 | `or` | `or` |
| 逻辑非 | `not` | `not` |
| HTTP GET | `http.request.method == "GET"` | `tcp port 80 and (tcp[...]:4] = 0x47455420)` |

**使用建议**：Wireshark 显示过滤器可随时修改、不会丢失数据，适合分析阶段灵活切换视角；tcpdump 的 BPF 过滤器在内核层执行，不符合条件的包不会被复制到用户空间，适合在高流量服务器上直接抓取目标流量或长期保存。两者配合使用效果最佳：用 tcpdump 在服务器上精准捕获并保存为 `.pcap`，下载后用 Wireshark 的显示过滤器深入分析。