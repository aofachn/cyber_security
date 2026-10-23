# Snort 规则介绍与工作模式

> Snort 是一款开源网络入侵检测与防御系统（IDS/IPS），核心能力在于实时流量分析、协议解析与内容匹配。它既可以像 tcpdump 一样做纯嗅探，也可以作为完整的入侵检测引擎，依据规则库识别攻击行为。

## 一、工作模式

Snort 有三种经典工作模式：

**嗅探器模式（Sniffer Mode）**：从网络读取数据包并作为连续流显示在终端上，类似 tcpdump，适合快速查看实时流量。

**数据包记录器模式（Packet Logger Mode）**：将数据包记录到硬盘，便于事后离线分析或取证。

**入侵检测模式（Intrusion Detection Mode）**：最复杂、可配置性最强的模式。Snort 将网络流量与规则集进行匹配，识别正常包与攻击包，并依据规则中定义的动作（alert、log、drop 等）做出响应。使用此模式需要正确配置规则。

## 二、Snort 规则结构

Snort 3 的规则由**规则头（Rule Header）** 和**规则体（Rule Body）** 两部分组成。

**规则头**定义“对什么样的流量做什么”，包含五个核心元素：
- **动作**：如 `alert`（告警）、`log`（记录）、`drop`（丢弃/阻断，IPS 模式）
- **协议**：如 `tcp`、`udp`、`icmp`、`ip`
- **源 IP 与源端口**
- **方向操作符**：`->`（单向）或 `<>`（双向）
- **目的 IP 与目的端口**

**规则体**位于括号内，定义“在什么条件下触发”，选项以分号分隔，关键字与参数之间用冒号连接。常用选项包括：
- `msg`：告警时显示的消息文本
- `content`：在载荷中搜索指定字符串/字节序列
- `flow`：限定流量方向与连接状态（如 `to_client,established`）
- `sid`：规则唯一标识符
- `rev`：规则修订版本号

一个完整的 Snort 3 规则示例如下：

    alert tcp $EXTERNAL_NET 80 -> $HOME_NET any
    (
        msg:"Attack attempt!";
        flow:to_client,established;
        file_data;
        content:"1337 hackz 1337",fast_pattern,nocase;
        service:http;
        sid:1;
    )

该规则的含义是：对从外部网络 80 端口发往内部网络任意端口的、已建立连接的 TCP 流量，检查文件数据中是否包含字符串“1337 hackz 1337”（不区分大小写），匹配则告警。

## 三、Snort 2 与 Snort 3 的关键差异

Snort 3 在规则语法上做了显著简化：

**规则语法统一**：Snort 2 的规则书写不一致且需要行转义（行尾反斜杠），Snort 3 支持任意空白符，规则可以跨行书写而无需转义。

**注释更丰富**：Snort 2 仅支持 `#` 注释；Snort 3 支持 `#`、`#begin/#end` 标记以及 C 语言风格的 `/* */` 注释。

**解析器更健壮**：Snort 3 的规则解析器提供更多可选警告，减少了因语法问题导致的静默失败。

**性能优化**：Snort 3 的快速模式匹配缓冲区从 6 个增加到 14 个，规则语法更简洁也使得运行速度更快。

## 四、规则示例（实用场景）

**SSH 暴力破解检测**（60 秒内同一源 5 次连接尝试）：

    alert tcp any any -> $HOME_NET 22 (
        msg:"IDS SSH Brute Force Attempt";
        threshold:type threshold, track by_src, count 5, seconds 60;
        sid:1000003;
    )

**SQL 注入尝试检测**（HTTP URI 中出现 `' OR 1=1 - `）：

    alert tcp any any -> any 80 (
        msg:"Possible SQL Injection Attempt";
        content:"' OR 1=1 - "; http_uri; nocase;
        sid:1000006;
    )

## 五、补充说明

Snort 规则中的 `content` 匹配默认是大小写敏感的，需要不区分大小写时加上 `nocase`。`fast_pattern` 用于指定该内容匹配作为快速模式匹配的锚点，可提升检测性能。在 IPS 模式下，动作 `drop` 才能真正阻断流量；在纯 IDS 模式下，通常使用 `alert` 或 `log`。