# OnePlus 13T Resources

一加 13T（`PKX110` / `OP60F5L1` / `pagani`）资料与实机研究记录。

本仓库不是“一键刷机包”，也不把论坛转述当成事实。目标是把设备资料、源码版本、实机测试、失败路径和恢复边界整理成可追溯、可复核的知识库。

> **资料收集状态：本轮收集已结束（2026-08-24）**
>
> 当前仓库转入整理、复核和实机验证阶段。已经筛选保留的资料继续按冻结提交与证据等级维护；未通过筛选的候选不写入文档或来源索引。结束收集不等于停止维护，既有结论仍可由新的实机证据或上游变更修正。

> **高风险提醒**
>
> 解锁、刷写、切换槽位、格式化、恢复分区均可能导致数据丢失或设备无法启动。仓库中的历史记录不等于针对另一台设备的操作授权；执行前必须重新确认设备、版本、槽位、文件哈希和回滚路径。

## 当前已知状态

最后核验：**2026-08-24**。

| 项目 | 已确认结论 | 证据等级 |
|---|---|---|
| 设备身份 | OnePlus 13T；`PKX110` / `OP60F5L1` / `pagani`；SM8750 `sun`；project `24821` | 实机确认 |
| Stock 历史基线 | `PKX110_15.0.2.302(CN01)`，Android 15；完整 OTA、payload 和已提取镜像完成哈希/签名校验 | 主机与实机确认 |
| 当前救援槽 | 2026-08-16 时 slot A 为可启动的 ColorOS `16.0.3.501(CN01)` | 实机确认 |
| 当前测试系统 | slot B 成功启动 LineageOS 23.2 非官方构建，Android 16，kernel 6.6.142 | 实机确认 |
| TWRP | `recovery_b` 上的 TWRP 3.7.1_16 已通过启动、触控、ADB、FBE 解密、MTP 读取、OTG 等非破坏性测试 | 实机确认 |
| 指纹 | UDFPS 录入仍失败；OPlus HAL 等待 UI-ready 约 502 ms 后以 vendor error `15001` 退出 | 实机日志确认 |
| 蜂窝网络 | NR SA、数据和通话链路正常；低信号格主要是 AOSP NR 阈值/CarrierConfig 映射问题 | 实机日志确认 |
| GApps 事件 | Google Clock 缺少两项 priv-app allowlist 权限造成 `system_server` 循环崩溃；随后 `UnInstall.zip` 流程移除了多项 Google 应用 | 实机日志确认 |
| EDL / ARB | `.501` 被公开固件分析列为 ARB 1；一组声称适用于 ARB1 的免授权 Firehose 引导已完成静态检查，但尚未在本机加载验证 | 来源交叉核对；设备未验证 |

完整状态见 [设备与版本基线](docs/device-baseline.md)、[LineageOS 记录](docs/lineageos-23.2.md) 和 [GApps 事件记录](docs/gapps.md)。

## 文档导航

### 实机基线与可复现记录

- [设备与版本基线](docs/device-baseline.md)
- [LineageOS 23.2 构建、安装与硬件状态](docs/lineageos-23.2.md)
- [TWRP 研究与实机验证](docs/twrp.md)
- [GApps / NikGapps 事件记录](docs/gapps.md)
- [备份与恢复边界](docs/backup-and-recovery.md)

### 高风险流程与社区参考

- [EDL、Firehose 与 ARB1 待验证引导](docs/edl-arb.md)
- [PKX110 转 CPH2723 OxygenOS 社区流程参考](docs/oxygenos-conversion.md)
- [OPlus / ColorOS 相机移植线索](docs/camera-porting-clue.md)

### 源码比较与候选目录

- [OnePlus 13T 设备树比较](docs/device-tree-comparison.md)
- [候选参考目录（未应用／未验证）](docs/pending-references.md)
- [AOSP 源码差异审计摘要](docs/aosp-gap-audit.md)

### 方法、数据与参与规则

- [记录方法、收集状态与证据等级](docs/methodology.md)
- [来源索引（机器可读）](data/source-index.yaml)
- [功能差异矩阵（TSV）](data/feature-matrix.tsv)
- [参与和提交规则](CONTRIBUTING.md)

## 仓库怎样记录信息

本仓库采用“**文档是事实库，Issue 是收件箱**”的模式：

1. `README.md` 只保留设备身份、关键警告和当前摘要。
2. `docs/` 保存经过整理的事实、实验时间线、失败结论和边界。
3. `data/source-index.yaml` 保存来源类别、URL、分支、冻结 commit、检查日期和可信状态。
4. `data/feature-matrix.tsv` 保存适配差异，方便脚本或表格工具处理。
5. Issue Form 用于提交纠错、测试结果或明确要求补充的新来源；仅有链接且未经筛选的信息不会直接写入事实库。
6. 每次更新通过 Git commit 留下变更原因；会变化的上游状态同时保留“当时冻结值”和“最近检查值”。

## 证据等级

| 标签 | 含义 |
|---|---|
| `DEVICE_VERIFIED` | 在明确记录的 PKX110 环境中实机观察并留有日志 |
| `HOST_VERIFIED` | 文件、源码、OTA、镜像或构建产物在主机侧完成解析/哈希/签名检查 |
| `SOURCE_VERIFIED` | 已直接读取上游源码、提交或官方文档 |
| `UPSTREAM_CLAIM` | 上游 README 或发布页声明，但本机尚未复测 |
| `HYPOTHESIS` | 有证据支持的解释，仍缺决定性验证 |
| `NOT_TESTED` | 明确没有测试；不得写成“支持”或“可用” |
| `SUPERSEDED` | 曾经正确，但已被后续测试或设备状态替代 |

## 资料边界

- 不上传 ColorOS OTA、专有 vendor blobs、Google APK、未经脱敏的个人日志、备份镜像、密钥或设备标识。
- 用户明确提供并要求归档的社区截图，只能作为有主题边界的上下文证据存入 `assets/community-clues/`；必须登记哈希和验证状态，不能单独证明功能可用。
- 大型二进制不进入 Git 历史；必要时只记录可信来源、文件名、版本和校验值。
- 不公开本机绝对路径、ADB 序列号、账户信息、SIM/Wi-Fi 信息或认证材料。
- CoolApk（酷安）内容默认排除；目前只为 `data/source-index.yaml` 中逐条列出的 ARB1/EDL、相机、指纹和显示 HDR 线索设立有范围、可撤销的例外。例外不扩展到相关账号的其他内容，也不会把社区声明提升为实机事实。
- 第三方仓库和文件各自遵循其原许可证；本仓库的链接不代表重新授权或背书。

## 封存后的验证队列

本轮主动外部资料收集已经结束。下面只安排对现有基线和已筛选参考的验证，不以继续堆积论坛链接为目标。

1. 以当前 Android 16 framework 为基线补齐 UDFPS `notify_fppress` / UI-ready 依赖链，并做回归测试。
2. 从 `.501` product/CarrierSettings 确认中国运营商的真实 NR 信号格阈值；不要通过更换 modem/firmware 解决 UI 映射问题。
3. 重新构建包含正确 Google Clock priv-app allowlist 的 GApps 方案，并以干净安装流程验证；临时 `log` 模式不能当永久修复。
4. 为蓝牙配对、GNSS 实际定位、相机实拍和长时间通话补充可复现的实机记录。
5. 对照 `oneplus13t-fix-camera` 的 ODM 文件清单与当前 Lineage vendor tree，区分“缺少设备配置/blob”和“缺少 OPlus framework”两类相机移植依赖。
6. 按[候选参考目录](docs/pending-references.md)和[设备树比较](docs/device-tree-comparison.md)固定的 commit，再分别审计构建补丁、UDFPS、显示配置与 2.4 GHz Wi-Fi 差异。
