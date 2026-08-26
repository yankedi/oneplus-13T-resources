# ARB1 免在线授权 9008（EDL）：Linux 实机验证报告

测试日期：2026-08-25 至 2026-08-26；整理日期：2026-08-26。时间均为主机 `Asia/Hong_Kong`（UTC+08:00）。

## 结论与适用范围

**本台 OnePlus 13T 的已安装 `xbl_config_b` 声明 ARB 1；指定三文件引导在 Linux 下无需 OPLUS 在线授权即可进入 Firehose，完成存储信息查询、受限读取，以及 `recovery_b` 的写入和完整回读验证。** 后续普通重启成功，再通过 Android ADB 进入了 Lineage Recovery。

这不是“任意分区可读写”或“可降级”的证明。首次写入曾因传输收尾异常失败，修正后第二次写入才成功；本报告保留失败过程。Recovery 仅确认启动身份、进程和 root ADB，显示、触控、解密及备份恢复没有完成验收。

- `DEVICE_VERIFIED`：实际通信、读取、两次写入尝试的不同结果、最终完整回读及后续启动。
- `HOST_VERIFIED`：文件哈希、GPT/镜像解析、AVB 校验、原引导指令的局部模拟和发送器回归。
- `SOURCE_VERIFIED`：冻结开源工具、Linux 传输路径及相关实现。
- `HYPOTHESIS`：首次失败的 ZLP 配置不匹配及 300 秒超时复位解释，证据高度吻合，但没有现场 USB 总线抓包或复位原因寄存器读数。
- `NOT_TESTED`：硬件熔丝/RPMB 回滚计数器、降级、ARB 绕过、其他分区写入和完整救砖。

本页是一次实验报告，不是可直接套用的刷机脚本。另一台设备或另一次会话必须重新确认身份、固件、槽位、存储几何、备份和授权范围。

## 1. 环境与材料

| 项目 | 本次记录及边界 |
|---|---|
| 设备 | OnePlus 13T，`PKX110` / `OP60F5L1` / `pagani`，SM8750 |
| 槽位 / 系统 | 操作前和普通重启后均为 `_b`；项目基线为 Android 16 / LineageOS 23.2 |
| Firmware | 实读 `xbl_config_b` 的有效内容与冻结 `.501` 镜像一致；不等于重新审计了整机全部 firmware |
| Bootloader / kernel | 历史基线为 unlocked、kernel 6.6.142；本次未重新做 fastboot 锁状态或完整内核清单检查 |
| 救援槽 A | 保留 2026-08-16 的 ColorOS `.501` / TWRP 基线；本轮未写入、未切换或重新启动验收 A |
| 主机 | NixOS 26.05，Linux 7.1.10，x86_64；原生 Linux 工具，没有运行 Windows 程序或 Wine |
| USB 传输 | `05c6:9008`，Linux `qcserial` / TTY；不能把结论直接推广到所有 libusb 后端 |
| 权限与监控 | 预先通过 `pkexec` 获得 root，复用控制会话；20 ms 轮询捕获 9008 后自动发送引导，无需人工抢时间 |
| Programmer | `SM8750_V1.0.05`，编译标记 `Jun 24 2024 19:26:24`，平台返回 `SM8750_C` |

### 引导与开源工具

本次重新核对了 [ARB1 三文件的大小和 SHA-256](edl-arb.md#收到的三文件材料)，全部匹配。它们已由先前提交 [`6f36c2b`](https://github.com/yankedi/oneplus-13T-resources/commit/6f36c2b3e4732c99b1ddbd8a7e83e7a8a4be820f) 归档于 `resources/edl/arb1-sm8750/`；本次报告不增加或修改二进制。实机成功不证明原始作者归属、再分发许可或整个供应链可信。

| 组件 | 冻结来源 / 实际文件标识 |
|---|---|
| `QSaharaServer` / `fh_loader` 基础源码 | [bm16ton/oplus12r-edl @ a6dceddd](https://github.com/bm16ton/oplus12r-edl/tree/a6dceddd2ff74eabf0566ddbbb72e50d98826874) |
| 入场顺序参考 | [OPLUS EDL Tool v2 @ 72999902](https://github.com/salokrwhite/OplusEdlTool/tree/72999902af09e99c702866e0086482dc42b82bbb)；只参考源码流程，没有执行其 Windows 程序 |
| `QSaharaServer` SHA-256 | `09ec9ee19a18a792c809d7792813953514fc2560e4b7260c9ae58ef6dccd584a` |
| 入场及首次写入所用 `fh_loader` SHA-256 | `b70932624deca5c5eb2953a19dabe7a76820168739eaef18bfe2b87acac4fe3c` |
| 重试事务所用修订 `fh_loader` SHA-256 | `35d1423b90a029905a8012888cbf8bfeef5463cc64cf0f9b60a136698bfc9378` |

本地适配复用现有 C 工具，仅增加入场/事务编排与必要的发送器修复，不是重写 Sahara/USB 协议：

1. 入场按 Sahara image 13 programmer → digest → `verify` → signature → `sha256init` → UFS configure 执行；没有登录或调用 OPLUS 在线授权服务。
2. `--skip_configure` 用于保留已建立的握手状态。`--skip_signed_digest_ack` **只用于 signature 原始数据阶段**，不跳过分区写入的最终 ACK。
3. 重试时，实际入场 XML 改为 `ZlpAwareHost="0"`；事务发送器另外修复 XML 分片累积，并设置大小与停滞边界。不能只在启用了 `--skip_configure` 的后续命令加参数，就假定设备配置已改变。
4. 重试仍用冻结的入场发送器配合修正后的 configure，单分区事务及后续保活使用修订发送器。本报告没有发布本地补丁或可复用刷机执行器，不能把上游原版当作已具备这些修复。

入场日志并非完全无警告：`sha256init` 先后出现 `verify signature failed`、`verify signature ok`、`verify passed.`，最终 ACK 并完成后续读取；configure 还出现 `Mode= Invalid value` / `EnableFlash not found`。以实际后续结果界定能力，不删去这些警告。

## 2. ARB 1 的实读证据

2026-08-25 23:30，先通过 Android KernelSU 只读提取 `xbl_config_b`；次日通过 EDL 读取同一完整分区，哈希一致。

| 范围 | 字节数 | SHA-256 |
|---|---:|---|
| 完整 `xbl_config_b` | 409600 | `a29081b7eb20752dee63efd21f6f984531e4560c153a09d115aab644736eda28` |
| 前 88 × 4096 字节，与冻结 `.501` payload 镜像同长 | 360448 | `7b21574f864dda61a2cfcd16c8759723bdefd2fd03130a82d3eeab71f9771abf` |

按 [Anti-Rollback Checker 的冻结提取流程](https://github.com/Bartixxx32/Oneplus-antirollchecker/tree/6088e36feb702925e21978aae2c3e030d148cfca)，独立编译 [arbextract @ 320efd96](https://github.com/koaaN/arbextract/tree/320efd96ded606b6bbf9c0c9d04ceb35caba32a6)，两种长度均解析为 OEM metadata `3.0`、ARB `1`。字节级交叉检查：ELF 的 HASH 段位于 `0x56000`，OEM metadata 位于 `0x56040`，三个小端 32 位值为 `3, 0, 1`。

这是“已安装镜像声明 ARB 1”的 `DEVICE_VERIFIED + HOST_VERIFIED` 结果，**不是直接读取熔丝/RPMB 中的硬件计数器**，也不能由此推断任意旧镜像可启动。此前“仅有公开 `.501` 跟踪数据、尚未独立提取”的状态已由这次窄范围证据更新。

## 3. 能力矩阵

| 项目 | 本次结果 | 证据与限制 |
|---|---|---|
| 无 OPLUS 在线授权进入 Firehose | PASS | `DEVICE_VERIFIED`；限指定设备、三文件和 Linux 适配流程 |
| UFS 信息 / GPT | PASS | 6 个 LUN、12 份主备 GPT、139 个物理分区条目；CRC、主备一致性、范围及无重叠检查通过 |
| 选定分区读取 | PASS / 非全量 | `xbl_config_b`、B 槽 vbmeta 系列、后续 recovery 与启动哨兵；不是全盘备份 |
| `super` / `devinfo` | PARTIAL | 只读 super 前 1 MiB 元数据、devinfo 首 4 KiB；后者全零，不能判断解锁状态 |
| EDL 读取 `misc` | FAIL：设备端拒绝 | 实际发送 read 后返回权限 NAK，不是主机预检查阻止发送 |
| 设备端 `getsha256digest` | FAIL | 返回 `Can't open sha256 handle 0`；未改认证状态强行重试，改用读回文件的主机哈希 |
| `recovery_b` 写入 | 首次 FAIL；重试 PASS | 全实验共两次 program 尝试，均仅针对同一分区；重试有最终 ACK、完整回读与 AVB 校验 |
| EDL 普通 reset → Android B | PASS | 实际 reset ACK、USB 转换、`_b` 和 `sys.boot_completed=1` |
| EDL 直接 reboot recovery | 未执行；当前解析分支未发现支持 | 静态审计不能代替实机模式测试；未知 power 值会落到普通 reset，不盲试 |
| Android ADB → Lineage Recovery B | PASS：启动身份及 root ADB | 不是 TWRP；UI、触控、解密、挂载和备份恢复未通过完整验收 |
| 擦除、其他分区写入、关机、reset_to_edl、降级 | NOT_TESTED | 命令表列出功能或权限位为 1，不等于安全可用 |

### 存储读取范围

UFS 返回三星 `KLUEG4RHHD-B0G1`，启动日志报告 UFS 4.0 / 256 GB，块大小 4096。LUN 0–5 分别为 `61706240, 2048, 2048, 3072, 634880, 130048` 个块。没有把 UFS boot-LUN 编号当成 Android A/B 槽号。

GPT 使用显式 `PrimaryGPT` / `BackupGPT` 标签，分别读取各 LUN 首 6 块、末 5 块。此前 helper 生成空标签并被拒绝为 `label cannot be null`，不代表 GPT 损坏。只读采集审计共识别 19 个 read 请求：18 个成功、`misc` 失败；零字节失败占位文件不算备份。

`getbasicdata` 的旧主机解析器在无属性标签上崩溃；改为带 `value="ping"` 的请求后成功。这是工具兼容性问题，不记成设备能力失败。入场阶段的默认 trace 曾被逐步覆盖，仅最终 configure trace 完整保留；其他入场阶段有各自日志与调用参数，不能声称全程原始字节 trace 都齐全。

## 4. 时间线：保留失败，区分阶段

| 时间（2026-08-26） | 已有日志事实 |
|---|---|
| 08:45–09:03 | 自动捕获并完成只读 UFS/GPT、引导元数据采集；会话在 XML 边界串行保活 |
| 09:40:22 | 独占端口，暂停保活，重新读取目标备份、GPT 与五项 B 槽启动哨兵 |
| 09:40:31–09:40:33 | 首次 program 获得初始 `ACK rawmode=true`；主机完成 100 × 1 MiB write，但没有最终 ACK |
| 09:41:32 | `fh_loader` 自行超时退出 1；事务停止，不再插入 XML、保活、重试或 reset |
| 09:45:33 / 09:45:46 | 末块发送约 300 秒后 9008 断开，与 programmer 超时逻辑吻合；随后 Android USB 返回并确认 B 槽启动 |
| 其后主机诊断 | 两次 Android 完整读回确认混合尾部；只分析既有证据、源码和隔离模拟，不向手机发测试命令 |
| 14:33:48 / 14:34:39 | 经授权重新布防，20 ms 自动捕获；本次实际 configure 为 `ZlpAwareHost=0` |
| 14:40:40–14:40:49 | 在确认的 XML 边界交接端口，完成新备份和检查后，重试一次 program |
| 14:40:52 / 14:41:00 | 收到最终 `ACK rawmode=false`；100 MiB 完整回读、非目标哨兵和 GPT 比较通过 |
| 14:41:01–15:06:14 | 每 25 秒串行只读保活；直到用户明确要求普通重启，停止 keeper 后发送 reset |
| 15:06:25 起 | 普通 Android USB 返回，确认 `_b`、normal boot、`sys.boot_completed=1` |
| 15:12:58 / 15:13:13 起 | 预检后发送一次 ADB reboot recovery；重新枚举并确认 Lineage Recovery B、root ADB |

## 5. 首次失败：不是简单“忘了保活”

首次会话实际声明了 `ZlpAwareHost=1`，而所用 Linux TTY 发送路径没有逐块发送 USB 零长度包（ZLP）。原生 `fh_loader` 的 Linux 默认值虽为 0，但显式入场 configure=1 已生效，后续 `--skip_configure` 不会自动纠正它。

主机检查了对应 Linux 7.1.10 的 `qcserial` / `usb_wwan` 源码，以及同一 programmer 中的 ARM64 接收与超时分支；使用 Capstone 5.0.7、pyelftools 0.33 和 Unicorn 2.1.4，在隔离内存中执行原指令并 stub 硬件接口：

- `ZlpAwareHost=0` 时收满 1 MiB 后上报完成；`ZlpAwareHost=1` 时在整包长度处继续等下一包。下一块的首包可推动前一块完成，最后一块却没有后续包。
- 默认传输等待超时为 300000 ms；局部模拟在 300001 ms 进入复位分支，测试在真实 MMIO 前截停。
- 实机末块发送到断开的间隔约 300 秒，与上述机制吻合；首次失败后的命令审计没有主机 reset。

这是高置信的事件解释，不是现场捕获了缺失 ZLP 或硬件复位原因。Android 的 `shutdown,userrequested` 属性也不能单独证明主机发送了 reset。传输仍可能处于 raw 数据阶段时，不恢复 XML 保活是必要的安全停止；盲发 NOP 可能被当成镜像字节，不能用来修复收尾。

首次失败后，100 MiB 实机镜像与候选只有 8 个字节不同，都在最后 64 字节的 AVB footer 内；该 footer 完整保留了旧 TWRP 的偏移 `58916864`，而新 Lineage 元数据位于 `18763776`。不能据此声称“只漏写 8 字节”：尾部大量共同零填充，使多种未提交块长度得到同一回读内容。

另一个独立缺陷是旧 `fh_loader` 不能可靠解析分片 XML ACK；它在 PTY 测试中复现，但首次实机失败阶段根本没有收到最终 ACK 字节，因此不是那次日志已证明的直接原因。重试同时修复分片累积，并保留最终 ACK 的强制校验。

重试前的 5 阶段入场 selftest、3 项 100 MiB PTY 测试、6 项分片/NAK/超大及停滞边界测试、16 项事务故障测试均按预期通过或安全停止。**这些是 `HOST_VERIFIED`，不是 USB、UFS 或设备断电故障注入测试。**

## 6. 重试成功的验证闭环

目标严格限定为 GPT 中的 `recovery_b`：LUN 4，LBA `545704–571303`，`25600 × 4096 = 104857600` 字节。采用 [2026-08-14 连贯 target-files 的 recovery](local-build-audit-2026-08-25.md)，没有使用混合的 active build 输出，也没有生成新镜像。

| 文件状态 | SHA-256 |
|---|---|
| 原始 TWRP，100 MiB，双次读取并保留第二物理介质副本 | `133e83e4e4fb4fb3d1d6f42b510d6b1685c84a6fc605cff0645d6718c1f52ccb` |
| 首次失败后混合镜像；重试写前另行完整备份 | `81e74741271aaa6783a6f3576591839d342d859fc6c93a7762fc82ccf99647ca` |
| Lineage 候选 = 重试 EDL 完整读回 = 后续 Android 设备侧哈希 | `e136fd6549526a92e2ac1c805c80d7762418448824f4e2d9ec11209dbfa9bef7` |

重试事务只发送一次 program，必须同时满足：

1. 初始 raw-mode ACK、全部 payload、最终 `ACK rawmode=false`，发送器退出 0。
2. 100 MiB 实机读回与候选逐字节及 SHA-256 相同；不是只看进度 100%。
3. 对**实际回读文件**验证 AVB 内容和 `SHA256_RSA4096` 签名。完整 1032 字节公钥与新读取的 `vbmeta_b` recovery chain 一致；rollback index `1785542400`、parent location `1` 是镜像元数据，不是硬件计数器。
4. `boot_b`、`vendor_boot_b`、`dtbo_b`、`init_boot_b`、`vbmeta_b` 五项完整哨兵，以及 LUN4 主备 GPT，写前/写后哈希和逐字节比较均不变。

上述比较只覆盖列出的区域，不能宣称对整盘每个字节做过不变性证明。没有向 A 槽、firmware、NV、userdata 发出写入，没有 erase、patch、格式化或切槽。原始 TWRP 回滚镜像保留，但本轮没有执行恢复 TWRP。

正常 XML 阶段的保活与事务共用独占端口锁，每 25–30 秒发送一次只读 `getstorageinfo`。交接必须等完整 ACK 和已知 XML 边界；不允许两个 sender 并行，更不允许 raw-mode 中夹入保活。该机制不保证断线、主机睡眠或进程终止后的连接仍存在。

## 7. `misc` 权限与重启模式边界

实际 read 请求针对 `misc` 的 LUN0、start sector `16392`、256 个 4096 字节块。脱敏后的关键响应为：

```text
entry:misc, w:1, r:0, e:1, c:0, s:0, start:0x4008, num:0x100
read on misc:16392:256 not allowed on external network
not get permission,line=146
response value=NAK, rawmode=false
```

拒绝来自手机端 programmer，发生在真正发送 read 后；主机 root、跳过主机预检查都不能把这个响应变成成功。`w:1` / `e:1` 只是返回的权限字段，不证明写入/擦除可行或安全。没有伪造 GPT 标签、改范围或修改引导绕过此限制。

已有公开对照是 [linux-msm/qdl issue #295](https://github.com/linux-msm/qdl/issues/295)：另一 SM8650 环境也出现 VIP 握手后 `not allowed on external network`。[参与者的讨论](https://github.com/linux-msm/qdl/issues/295#issuecomment-5056020313)将 VIP 与 programmer 分区访问限制分开；该讨论不是本机或本组三文件的 OEM 证明，其中利用 GPT 标签的猜测也没有可据以标为本机成功的结果。访问限制还可能涉及范围等条件，不能把另一例中的失败一概归结为同一权限位。这里只记录“有同类公开现象”，不泛化成所有 OPLUS 引导必然如此。

对当前引导的 power 分支静态检查只找到 `reset`、`off`、`reset_to_edl`；未知值会回落普通 reset，未发现直接 recovery 选择。`setbootablestoragedrive` 也不能当成 Android 槽位或 Recovery 选择器。本机只实测了普通 reset，其余模式不因静态分支存在就记为 PASS。

用户随后明确要求重启，才在 XML 边界停止 keeper 并发送一次 `<power value="reset"/>`，收到 ACK，之后确认 Android B 启动。进入 Recovery 使用的是后续单独授权的 Android ADB 重启，不是直接 EDL 跳转。

## 8. 启动验收：当前是 Lineage Recovery，不是 TWRP

Android 重启 Recovery 前确认 B 槽、电池 90% 且充电、`recovery_b` 哈希仍为上述 Lineage 候选。通过 Android root 另行备份了完整 1 MiB `misc`：这是 **ADB 通道读取成功，不是 EDL 权限绕过**。BCB 前 2048 字节全零，未发现 cache recovery command、openrecoveryscript 或 extendedcommand 等待执行文件。

一次 ADB reboot recovery 后，属性和进程给出：

```text
ro.boot.slot_suffix=_b
ro.bootmode=recovery
ro.lineage.version=23.2-20260814-UNOFFICIAL-pagani
ro.twrp.version=
ro.twrp.boot=
init.svc.recovery=running
sys.usb.config=adb
sys.usb.state=adb
```

ADB 状态为 recovery，shell `uid=0`，Recovery 进程在采样窗口内稳定，`/sbin/twrp` 不存在。USB 重新枚举本身不等于系统循环重启，也不按 USB ID 数据库名称把它误判为 fastboot。

Recovery 日志同时出现 `Atomic Commit failed`、`Invalid f2fs superblock` 和 `Failed to mount /data`。这些问题尚未定位；没有屏幕截图或触控验收，不能宣布显示、解密和挂载正常，也不能仅凭日志断言用户数据损坏。没有执行 fsck、手工挂载、wipe、格式化、sideload 或再次刷写。

Recovery 启动时自行写入/清除 BCB 是正常启动行为，因此“本阶段没有主动刷写”不等于设备存储绝对零变化。最后一次观察时手机留在 Lineage Recovery，原 EDL keeper 已停止；这是一份时间截面，不是持续在线保证。此前 TWRP 的历史非破坏性 PASS 仍保留于 [TWRP 记录](twrp.md)，不能把它当作当前 B 槽内容。

## 9. 证据留存与未覆盖项

原始证据在本地按会话分别封存，不随这次文档提交公开：

| 证据组 | 留存内容 |
|---|---|
| ARB / 只读采集 | Android 与 EDL 镜像、解析输出、UFS/GPT/LP 审计、实际命令审计及捕获 SHA-256 清单 |
| 首次失败与诊断 | 初始/最终 ACK 观察、USB 时间线、两次 Android 回读、失败事务审计、原指令测试及文件哈希 |
| 重试成功 | `success-audit.json`、写前/写后读回与 GPT 审计、AVB 回读验证、实际 XML trace；静态证据清单共 151 项，活动保活日志另存 |
| 普通重启 / Recovery | reset ACK、USB 事件、启动属性、Recovery 进程及日志、重启前 misc 私有备份 |

公开内容限本页摘要、可复核的文件指纹、冻结源码链接及关联状态；不上传设备序列号、原始日志、备份镜像、主机私人路径、账户或认证材料。独立第三方无法只靠本页重新验证未公开原始证据的全部细节；`DEVICE_VERIFIED` 表示本项目留有实测证据，不是多台设备的第三方复现。

硬件 ARB 计数器、任意分区权限、完整闪存备份、断电恢复、其他固件版本/机型、Recovery 全功能和降级救砖仍为 `NOT_TESTED`。继续测试不能复用本次一次性事务执行器或假定旧备份、旧槽位表永远有效；见 [备份与恢复边界](backup-and-recovery.md)。
