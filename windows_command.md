### 系统管理与核心工具

| 命令 | 功能说明 |
| :--- | :--- |
| **`wf.msc`** | **高级安全 Windows Defender 防火墙**，用于精细配置入站/出站规则[citation:4][citation:9] |
| `compmgmt.msc` | 计算机管理（集成事件查看器、磁盘管理、服务等）[citation:6] |
| `services.msc` | 服务管理器 |
| `devmgmt.msc` | 设备管理器[citation:6] |
| `diskmgmt.msc` | 磁盘管理[citation:6] |
| `eventvwr.msc` | 事件查看器[citation:2][citation:6] |
| `taskmgr` | 任务管理器 |
| `perfmon` / `perfmon.msc` | 性能监视器[citation:6][citation:8] |
| `resmon` | 资源监视器[citation:6] |
| `msconfig` | 系统配置（启动项、引导）[citation:6] |
| `mmc` | 打开空的微软管理控制台，可自行添加管理单元[citation:6] |
| `sysdm.cpl` | 系统属性（计算机名、环境变量、性能选项）[citation:11] |
| `cleanmgr` | 磁盘清理工具[citation:6][citation:8] |
| `dfrgui` | 优化驱动器（碎片整理/TRIM）[citation:6] |
| `MdSched` | Windows 内存诊断程序[citation:6][citation:8] |
| `chkdsk.exe` | 磁盘检查（建议在管理员命令提示符下运行）[citation:6] |
| `dxdiag` | DirectX 诊断工具[citation:6] |
| `msinfo32` | 系统信息汇总[citation:6] |
| `sigverif` | 文件签名验证程序[citation:8] |
| `rsop.msc` | 组策略结果集（查看实际生效的策略）[citation:8] |

### 安全与账户策略

| 命令 | 功能说明 |
| :--- | :--- |
| `secpol.msc` | 本地安全策略（密码策略、审核策略、用户权限分配） |
| `gpedit.msc` | 本地组策略编辑器（家庭版默认无）[citation:6] |
| `certmgr.msc` | 证书管理器（管理用户/计算机证书）[citation:6] |
| `lusrmgr.msc` | 本地用户和组[citation:6] |
| `azman.msc` | 授权管理器 |
| `tpm.msc` | 受信任的平台模块（TPM）管理 |
| `syskey` | Windows 系统安全工具（SAM 数据库加密，较旧系统） |

### 网络与共享

| 命令 | 功能说明 |
| :--- | :--- |
| `ncpa.cpl` | 网络连接（网卡配置）[citation:6] |
| `firewall.cpl` | Windows 防火墙基础面板[citation:6] |
| `fsmgmt.msc` | 共享文件夹管理[citation:6] |
| `shrpubw` | 创建共享文件夹向导[citation:8] |
| `iscsicpl` | iSCSI 发起程序 |
| `WF.msc` | （同 wf.msc）高级防火墙 |
| `inetcpl.cpl` | Internet 属性（代理、安全区域）[citation:6] |

### 控制面板与系统设置（.cpl 项）

| 命令 | 功能说明 |
| :--- | :--- |
| `appwiz.cpl` | 程序和功能（卸载软件）[citation:6] |
| `powercfg.cpl` | 电源选项[citation:6] |
| `main.cpl` | 鼠标属性[citation:6] |
| `mmsys.cpl` | 声音设置[citation:6] |
| `desk.cpl` | 屏幕分辨率/显示设置[citation:6] |
| `intl.cpl` | 区域和语言[citation:6] |
| `hdwwiz.cpl` | 添加硬件向导[citation:6] |
| `control admintools` | 打开“管理工具”文件夹 |
| `OptionalFeatures` | 打开“启用或关闭 Windows 功能”对话框[citation:6][citation:8] |

### 组件服务与诊断

| 命令 | 功能说明 |
| :--- | :--- |
| `dcomcnfg` | 组件服务（COM+ 配置）[citation:6] |
| `comexp.msc` | 组件服务（同上）[citation:6] |
| `msdt` | Microsoft 支持诊断工具[citation:6] |
| `psr` | 问题步骤记录器（记录操作复现步骤）[citation:6] |
| `printmanagement.msc` | 打印管理[citation:6] |
| `odbcad32` | ODBC 数据源管理器[citation:6] |
| `wmimgmt.msc` | WMI 控件（Windows 管理规范） |

### 远程与实用工具

| 命令 | 功能说明 |
| :--- | :--- |
| `mstsc` | 远程桌面连接[citation:6][citation:8] |
| `msra` | Windows 远程协助[citation:6] |
| `osk` | 屏幕键盘[citation:6][citation:8] |
| `magnify` | 放大镜[citation:6][citation:8] |
| `narrator` | 讲述人（屏幕朗读）[citation:6][citation:8] |
| `explorer` | 打开资源管理器[citation:6] |
| `logoff` | 注销当前用户[citation:6][citation:8] |
| `shutdown -s -t 秒数` | 定时关机；`shutdown -a` 取消[citation:6] |
| `rundll32 user32.dll,LockWorkStation` | 锁定计算机[citation:6] |