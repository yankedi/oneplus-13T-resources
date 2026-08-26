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

最后更新：**2026-08-26**。各项核验日期和适用范围以对应记录为准。

| 项目 | 已确认结论 | 证据等级 |
|---|---|---|
| 设备身份 | OnePlus 13T；`PKX110` / `OP60F5L1` / `pagani`；SM8750 `sun`；project `24821` | 实机确认 |
| Stock 历史基线 | `PKX110_15.0.2.302(CN01)`，Android 15；完整 OTA、payload 和已提取镜像完成哈希/签名校验 | 主机与实机确认 |
| 当前救援槽 | 2026-08-16 时 slot A 为可启动的 ColorOS `16.0.3.501(CN01)`；同版本公开完整 OTA 已完成整体、payload 与选定分区哈希审计 | 实机确认 + 主机确认 |
| 当前测试系统 | slot B 为 LineageOS 23.2 非官方构建 / Android 16；历史 kernel 6.6.142，2026-08-26 再次确认普通 Android B 启动完成 | 实机确认；未重测全部功能 |
| Recovery / TWRP | 2026-08-26 已通过 EDL 将 `recovery_b` 换为 Lineage Recovery；完整回读、启动及 root ADB 通过，UI/解密等待验收。TWRP 验收保留为历史记录 | 实机确认 |
| 指纹 | UDFPS 录入仍失败；OPlus HAL 等待 UI-ready 约 502 ms 后以 vendor error `15001` 退出 | 实机日志确认 |
| 蜂窝网络 | NR SA、数据和通话链路正常；低信号格主要是 AOSP NR 阈值/CarrierConfig 映射问题 | 实机日志确认 |
| GApps 事件 | Google Clock 缺少两项 priv-app allowlist 权限造成 `system_server` 循环崩溃；随后 `UnInstall.zip` 流程移除了多项 Google 应用 | 实机日志确认 |
| EDL / ARB | 实读 `xbl_config_b` 声明 ARB 1；Linux 下免在线授权进入 Firehose、UFS/GPT 查询、受限读取及 `recovery_b` 写入/完整回读均已验证 | 实机与主机确认；权限及降级边界见报告 |

完整状态见 [设备与版本基线](docs/device-baseline.md)、[LineageOS 记录](docs/lineageos-23.2.md) 和 [GApps 事件记录](docs/gapps.md)。

## 文档导航

### 实机基线与可复现记录

- [设备与版本基线](docs/device-baseline.md)
- [LineageOS 23.2 构建、安装与硬件状态](docs/lineageos-23.2.md)
- [2026-08-25 本地源码与构建产物审计](docs/local-build-audit-2026-08-25.md)
- [2026-08-26 ARB1 免在线授权 EDL：Linux 实机验证报告](docs/edl-arb1-device-validation-2026-08-26.md)
- [TWRP 研究与实机验证](docs/twrp.md)
- [GApps / NikGapps 事件记录](docs/gapps.md)
- [备份与恢复边界](docs/backup-and-recovery.md)

### 高风险流程与社区参考

- [EDL、Firehose 与 ARB1 引导的验证边界](docs/edl-arb.md)
- [PKX110 转 CPH2723 OxygenOS 社区流程参考](docs/oxygenos-conversion.md)
- [OPlus / ColorOS 相机移植线索](docs/camera-porting-clue.md)

### 源码比较与候选目录

- [OnePlus 13T 设备树比较](docs/device-tree-comparison.md)
- [`.501` 原厂包与现有设备树交叉审计](docs/stock-501-cross-audit.md)
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
- 既有提交 `6f36c2b` 已归档指定 ARB1 三文件至 `resources/edl/arb1-sm8750/`；这是限定的既有归档，不扩展到其他二进制或认证材料，也不代表其原始来源和再分发许可已获证明。本次实机报告仅更新文档与索引。
- 不公开本机绝对路径、ADB 序列号、账户信息、SIM/Wi-Fi 信息或认证材料。
- CoolApk（酷安）内容默认排除；目前只为 `data/source-index.yaml` 中逐条列出的 ARB1/EDL、相机、指纹和显示 HDR 线索设立有范围、可撤销的例外。例外不扩展到相关账号的其他内容，也不会把社区声明提升为实机事实。
- 第三方仓库和文件各自遵循其原许可证；本仓库的链接不代表重新授权或背书。

## 封存后的验证队列

本轮主动外部资料收集已经结束。下面只安排对现有基线和已筛选参考的验证，不以继续堆积论坛链接为目标。

1. 保留当前 SensorProps 与已存在的 stock init／通用 SELinux 访问链，定向移植 `OplusFodShim` 事件桥；先用节点上下文、AVC、加载证明和完整 UDFPS 场景做回归。
2. `.501` 未给出可直接复制的中国 NR 门限；继续追踪运行时 CarrierConfig 选择与 framework 行为，不把日本运营商、IMS RTP 或 LTE 属性表当成答案，也不更换 modem/firmware。
3. 重新构建包含正确 Google Clock priv-app allowlist 的 GApps 方案，并以干净安装流程验证；临时 `log` 模式不能当永久修复。
4. 为蓝牙配对、GNSS 实际定位、相机实拍和长时间通话补充可复现的实机记录。
5. 相机 ODM 模块 101/101 哈希一致、当前清单 916/916 路径存在；后续重点转为 HDR、变焦、录像等实机功能矩阵，而不是整批换 blob。
6. 2.4 GHz Wi-Fi 候选的两个配置已在 `.501` 找到，但普通加载路径未闭合；先做分频段复现并记录 persist 配置来源，再决定是否采用 `9280681`。
