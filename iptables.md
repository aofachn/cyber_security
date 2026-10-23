# iptables 常用操作示例

> 核心原则：**规则自上而下匹配，命中即停**，因此精确规则应放在宽泛规则之前。

## 一、查看与清空规则

    # 查看 filter 表规则（带行号、数字 IP、命中计数）
    iptables -L -n -v --line-numbers

    # 查看 nat 表规则
    iptables -t nat -L -n -v --line-numbers

    # 备份当前规则
    iptables-save > /tmp/iptables.bak

    # 清空所有规则（生产环境慎用）
    iptables -F
    iptables -X
    iptables -Z

## 二、基础放行与拦截

    # 放行 SSH（22）、HTTP（80）、HTTPS（443）
    iptables -A INPUT -p tcp --dport 22 -j ACCEPT
    iptables -A INPUT -p tcp --dport 80 -j ACCEPT
    iptables -A INPUT -p tcp --dport 443 -j ACCEPT

    # 放行回环接口
    iptables -A INPUT -i lo -j ACCEPT

    # 放行已建立连接的回包（避免重复检查）
    iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

    # 拦截单个 IP
    iptables -A INPUT -s 192.168.1.100 -j DROP

    # 拒绝并返回 ICMP 拒绝消息（对端立即感知）
    iptables -A INPUT -s 10.0.0.5 -j REJECT

## 三、默认策略

> 先放行 SSH，再设 DROP，防止自锁。

    iptables -A INPUT -p tcp --dport 22 -j ACCEPT
    iptables -P INPUT DROP
    iptables -P FORWARD DROP
    iptables -P OUTPUT ACCEPT

## 四、删除与插入规则

    # 按行号删除 INPUT 链第 3 条
    iptables -D INPUT 3

    # 按规则内容删除
    iptables -D INPUT -p tcp --dport 80 -j ACCEPT

    # 在第 1 位插入规则（优先级最高）
    iptables -I INPUT 1 -p tcp --dport 443 -j ACCEPT

## 五、限制速率与防暴力破解

    # 限制 80 端口每秒最多 10 个新连接（防 DoS）
    iptables -A INPUT -p tcp --dport 80 --syn -m limit --limit 10/s -j ACCEPT
    iptables -A INPUT -p tcp --dport 80 --syn -j DROP

    # 限制单 IP 60 秒内最多 5 次 SSH 新连接（防暴力破解）
    iptables -A INPUT -p tcp --dport 22 --syn -m recent --name ssh_brute --rcheck --seconds 60 --hitcount 5 -j DROP
    iptables -A INPUT -p tcp --dport 22 --syn -m recent --name ssh_brute --set -j ACCEPT

## 六、端口转发（NAT）

    # 开启内核转发
    echo 1 > /proc/sys/net/ipv4/ip_forward

    # 将外部 8080 转发到内网 192.168.1.10:80
    iptables -t nat -A PREROUTING -p tcp --dport 8080 -j DNAT --to-destination 192.168.1.10:80

    # 放行转发流量
    iptables -A FORWARD -p tcp -d 192.168.1.10 --dport 80 -j ACCEPT

    # SNAT / 共享上网（出口 MASQUERADE）
    iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

## 七、保存与恢复（重启不丢失）

    # Debian/Ubuntu（需安装 iptables-persistent）
    iptables-save > /etc/iptables/rules.v4
    iptables-restore < /etc/iptables/rules.v4

    # RHEL/CentOS
    iptables-save > /etc/sysconfig/iptables
    iptables-restore < /etc/sysconfig/iptables

## 八、补充说明

- **`DROP` 与 `REJECT` 的区别**：`DROP` 静默丢弃，对端表现为超时；`REJECT` 返回拒绝消息，对端立即感知。
- **自锁风险**：远程 SSH 操作时，设置 `INPUT DROP` 默认策略**之前**，务必先放行 SSH 端口，否则会锁死自己。
- **CentOS 7+ 注意**：系统默认使用 `firewalld`，若发现规则不生效，先确认 `firewalld` 是否在运行并干扰了 iptables。