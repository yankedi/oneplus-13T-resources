# 待使用／未验证参考

更新时间：2026-08-24

本页是后续构建、适配和排障的候选资料清单，不是已验证教程。除特别注明外，条目仅完成链接登记或社区线索归档，均不能据此宣称 OnePlus 13T（`PKX110` / `pagani`）已经支持相应功能。

## 状态与使用边界

| 状态 | 含义 |
|---|---|
| `LINK_CHECKED` | 链接在登记时可访问；不代表代码已经审计 |
| `COMMUNITY_CLUE` | 来自用户提供的社区文本或截图，只用于提出验证方向 |
| `NOT_REVIEWED` | 尚未逐文件阅读、比对许可证或固定 commit |
| `NOT_TESTED` | 尚未在本仓库记录的 PKX110 环境中测试 |
| `ALREADY_REFERENCED` | 已在其他文档引用，此处只纳入统一入口 |

使用任何源码前，应先固定 commit，并与[成功构建所用冻结版本](lineageos-23.2.md#冻结源码版本)做差异审计。网盘包、ROM 和 recovery 镜像还应另行核对来源、签名或哈希、适用固件、ARB 状态和恢复路径。

## 构建、设备树与硬件层

| 参考 | 计划用途 | 当前状态 | 边界 |
|---|---|---|---|
| [`ABNOTF/patches_for_pagani@1818541`](https://github.com/ABNOTF/patches_for_pagani/tree/1818541189e505057f84a34ad4ba86f86e571fb5) | 构建指南、UDFPS SystemUI 补丁和 Aperture 相机补丁参考 | `SOURCE_VERIFIED + NOT_APPLIED + NOT_TESTED` | 仓库不是设备树；指南指定的两棵实际源码输入已拆分登记在下两行 |
| [`ABNOTF/android_device_oneplus_pagani@560f47e`](https://github.com/ABNOTF/android_device_oneplus_pagani/tree/560f47ecd11bb520db4319bde01c0fa03c36fa50) | `patches_for_pagani` 指定的 `device/oneplus/pagani` 构建输入 | `SOURCE_VERIFIED + NOT_APPLIED + NOT_TESTED` | `lineage-23.2`；是 Development 冻结基线 `dfe9aa4` 的后代，指南引用不等于本仓库构建验证 |
| [`ABNOTF/android_device_oneplus_sm8750-common@5018caf`](https://github.com/ABNOTF/android_device_oneplus_sm8750-common/tree/5018cafefeb84ddc7ad1beda21f6d70ed6f28163) | `patches_for_pagani` 指定的 `device/oneplus/sm8750-common` 构建输入 | `SOURCE_VERIFIED + NOT_APPLIED + NOT_TESTED` | `lineage-23.2`；与 LineageOS common 冻结基线已分叉，指南引用不等于本仓库构建验证 |
| [`zzkeier/android_device_oneplus_pagani@f935f38`](https://github.com/zzkeier/android_device_oneplus_pagani/tree/f935f38673bc581a09944ec342984816c7db78ca) | `lineage-23.0` 机型设备树实现参考 | `SOURCE_VERIFIED + NOT_APPLIED + NOT_TESTED` | 与本仓库 `lineage-23.2` 成功构建基线不是同一分支 |
| [`zzkeier/android_device_oneplus_sm8750-common@4247c19`](https://github.com/zzkeier/android_device_oneplus_sm8750-common/tree/4247c19bb791f9ff293e0e169380c83a46c10bd2) | common 设备树和 2.4 GHz Wi-Fi 候选差异的分支级参考 | `SOURCE_VERIFIED + NOT_APPLIED + NOT_TESTED` | `lineage-23.2`；包含下一行的 `9280681`，但整棵树还有其他独有提交 |
| [`sm8750-common@9280681`：pagani 2.4 GHz Wi-Fi 修复](https://github.com/zzkeier/android_device_oneplus_sm8750-common/commit/92806812c82f10d42ea663c1b6348c2a97294d7b) | 精确比对 Wi-Fi 固件搜索路径与新增的 ODM 配置文件条目 | `SOURCE_VERIFIED + NOT_APPLIED + NOT_TESTED` | fork 中的候选 commit；尚未进入本仓库冻结的 LineageOS common tree |
| [`renhiyama/android_device_oneplus_pagani@04be737`](https://github.com/renhiyama/android_device_oneplus_pagani/tree/04be737a8349ffe940addaa75f3ea8b7140c1fc6) | XDA A1 公开的 `pagani` 机型树，定向参考 UDFPS 和区域适配 | `SOURCE_VERIFIED + NOT_APPLIED + NOT_TESTED` | `op13s`；只收源码，不把 A1 ROM 的旧测试结果外推到当前构建 |
| [`renhiyama/android_device_oneplus_sm8750-common@b83096a`](https://github.com/renhiyama/android_device_oneplus_sm8750-common/tree/b83096a7331d33a5732f38eb4bbfc4b781f54d87) | XDA A1 公开的 common 树，定向比较指纹 enrollment shim 与 touchDaemon SELinux 规则 | `SOURCE_VERIFIED + NOT_APPLIED + NOT_TESTED` | 源码注释声称处理解锁状态下新指纹录入；尚未在当前设备验证 |
| [`renhiyama/vendor_oplus_camera@563c432`](https://github.com/renhiyama/vendor_oplus_camera/tree/563c432ecd67c645cbc9ed20de7959d6173f5d34) | XDA A1 的 OPlus Camera vendor 源码入口 | `SOURCE_INSPECTED + NOT_APPLIED + NOT_TESTED` | 含 APK、库和其他 proprietary 内容；只登记链接，不复制二进制，且不是当前指纹问题的优先输入 |
| [`Neveark/android_device_oneplus_sm8750-common@86ca73e`](https://github.com/Neveark/android_device_oneplus_sm8750-common/tree/86ca73eaa40b6d06e2f9fd2a4336fd562aa101c9) | 用户筛选保留的 A3 common 树，比较指纹传感器参数 shim 及其他公共层差异 | `SOURCE_VERIFIED + NOT_APPLIED + NOT_TESTED` | 当前只取得 common 树，不推定可与任意 `pagani` 树直接组成正确构建 |
| [RandomLemon/android_hardware_oplus](https://github.com/RandomLemon/android_hardware_oplus) | OPlus hardware、UDFPS/FOD shim 候选实现参考 | `LINK_CHECKED + NOT_REVIEWED + NOT_TESTED` | 截图与仓库的关联由资料提供者给出；尚未定位决定性 commit 或文件 |
| [LineageOS 官方 OnePlus 13（dodge）构建指南](https://wiki.lineageos.org/devices/dodge/build/) | 参考 LineageOS 官方构建环境、同步、专有文件提取和编译流程 | `LINK_CHECKED + NOT_APPLIED_TO_PAGANI` | `dodge` 是 OnePlus 13，不是 OnePlus 13T 的 `pagani`；不能照搬设备命令、分区或专有文件 |

LineageOS Wiki 当前没有检索到 `pagani` 的官方设备构建页。因此，`dodge` 页面只能作为同代设备的流程示例，实际 manifest、lunch target、proprietary files 和分区配置仍以 pagani 源码为准。

上述 Development、zzkeier、ABNOTF 三套核心机型层／公共层的冻结 HEAD、依赖拓扑、文件数量、Git 分叉、显示、UDFPS、音频、power、proprietary 和 Wi-Fi 差异，以及 renhiyama／Neveark 的指纹定向比较，见 [OnePlus 13T 设备树比较](device-tree-comparison.md)。其中“ABNOTF 实际构建树”只表示构建指南的 clone 命令直接指定了这两个仓库，不表示本仓库已经用它们成功构建或完成设备测试。

XDA A2 未找到可供复用的公开设备树或构建源码，因此没有把该 ROM 本身加入源码候选。此前由用户单独提供的 DerpFest root／隐藏文字仍保留在下方，只作为工具线索，不作为 A2 开源实现。

## 2.4 GHz Wi-Fi / common tree 候选修复

候选来源是 [`zzkeier/android_device_oneplus_sm8750-common@9280681`](https://github.com/zzkeier/android_device_oneplus_sm8750-common/commit/92806812c82f10d42ea663c1b6348c2a97294d7b)，commit 标题为 `Fixing Pagani device performance on 2.4 GHz Wi-Fi`。本次直接读取到两处改动：

1. `init/ueventd.qcom.rc` 将 firmware 搜索路径从仅 `/vendor/firmware_mnt/image/` 扩展为 `/mnt/vendor/persist/copy/`、`/odm/etc/wifi/`、`/mnt/vendor/persist/` 和原 vendor 路径。
2. `proprietary-files.txt` 增加 `odm/vendor/etc/wifi/WCNSS_qcom_cfg_roam.ini` 与 `odm/vendor/etc/wifi/WCNSS_qcom_cfg_cmcc.ini`。

本仓库成功构建时冻结的 [`LineageOS/android_device_oneplus_sm8750-common@44ad18f`](https://github.com/LineageOS/android_device_oneplus_sm8750-common/commit/44ad18fa12b51983a36fb7ec67c54a6b4c032859) 仍只有原 vendor firmware 路径和基础 `WCNSS_qcom_cfg.ini`；截至 2026-08-24，LineageOS 的 `lineage-23.2` HEAD 仍是该 commit，没有上述两项变化。候选 fork 的 `lineage-23.2` HEAD 为 `4247c19bb791f9ff293e0e169380c83a46c10bd2`，包含该修复。

Neveark 历史中的 [`cb33454`](https://github.com/Neveark/android_device_oneplus_sm8750-common/commit/cb33454925ab2bec9eb4077a1cfeea4b9b86f3fa) 是同一两文件改动在另一基线上的移植，不重复列为第二条修复；其后 [`4106c3d`](https://github.com/Neveark/android_device_oneplus_sm8750-common/commit/4106c3dba07896445fcbe251879e3143c6964847) 又移除两个额外 WCNSS proprietary 条目。排重和冻结 HEAD 状态见[设备树比较](device-tree-comparison.md#24-ghz-提交排重)。

现有冒烟测试中的 Wi-Fi `PASS` 只记录了约 500 Mbps 的 L2 吞吐，没有单独记录频段、信道、配置文件加载路径或 2.4 GHz 复测，因此既不能证明当前构建存在该问题，也不能证明候选 commit 已修复本机。采用前应先确认 stock/ODM 中两个新增文件真实存在，再做最小 diff 构建，并分别记录 2.4 GHz 与 5 GHz 的关联、吞吐、热点和重连结果。

## 指纹 / UDFPS 线索

### OplusFodShim 与 hardware/oplus

用户提供的讨论截图中，`INT16` 将其方案概括为 `OplusFodShim`，并称调整后 AOD、熄屏指纹正常；同一讨论还提到用该方案替换另一套实现。该说法目前只记为 `COMMUNITY_CLUE + NOT_TESTED`，候选代码入口是 [RandomLemon/android_hardware_oplus](https://github.com/RandomLemon/android_hardware_oplus)。

![关于 OplusFodShim、AOD 和熄屏指纹的社区讨论](../assets/community-clues/2026-08-24-fingerprint-oplusfodshim.jpg)

截图里另有“相机在 Aperture 开启 HDR、放大超过 2.1 倍后可能冻结并崩溃”的旁支描述。它与下面的“显示 HDR 配置”不是同一个问题，本页不据此建立相机结论。

### `notify_fppress` 事件桥接

另一张截图中，`Synecdoche` 将指纹修复思路概括为：监测每次手指按下事件，并向 `/sys/kernel/oplus_display/notify_fppress` 写入 `1`。这与现有 [LineageOS UDFPS 根因边界](lineageos-23.2.md#udfps-根因边界)中的待查节点相符，但仍缺少完整实现和本机回归证据。

![关于 notify_fppress 节点的社区回复](../assets/community-clues/2026-08-24-fingerprint-notify-fppress.jpg)

后续若采用这条线索，至少还要核对 finger-down/up 生命周期、节点权限、ueventd、SELinux、HBM、AOD/熄屏路径，以及 `session.onUiReady()` 是否真实到达 HAL；不能只凭一次写入就标记指纹为修复。

## 显示 HDR 线索

用户提供的跨设备 SM8750 讨论中，`Synecdoche` 建议在官方包解压目录中搜索 `display_config`，认为 HDR 显示相关定义通常位于其中。

![关于从官方包查找 display_config 的社区回复](../assets/community-clues/2026-08-24-hdr-display-config.jpg)

该讨论的提问设备不是 OnePlus 13T，且截图没有原帖 URL、目标文件路径或 patch，因此状态为 `COMMUNITY_CLUE + NOT_TESTED`。后续只把它作为比对 `.501` 产品配置、display HAL/mapper 和当前 device tree 的搜索入口。

## 系统转换与固件档案

| 参考 | 计划用途 | 当前状态 | 边界 |
|---|---|---|---|
| [XDA：PKX110 转 CPH2723 OxygenOS](https://xdaforums.com/t/rom-pkx110-oxygenos-convert-oneplus-13t-to-13s.4743853/) | A5 的转换拓扑、分区角色、更新限制和风险案例 | `COMMUNITY_GUIDE + NOT_TESTED` | 已整理为[独立索引](oxygenos-conversion.md)；不是刷机建议，未下载或重传论坛文件 |
| [`spike0en/oneplus_archive@99e7002`](https://github.com/spike0en/oneplus_archive/tree/99e7002f229e2b6f817bbaaf873d5875f8c8e980) | PKX110 ColorOS／CPH2723 OxygenOS stock OTA 与分区镜像档案入口 | `SOURCE_VERIFIED_INDEX + ARTIFACTS_NOT_AUDITED` | 社区项目声称文件来自 OEM 且未修改；下载后仍需逐文件核对版本、SHA-256、区域与 ARB，不能把归档标签当成刷写许可 |

冻结源码把 `pagani` 映射到 `PKX110` 和 `CPH2723`，并分别列出 boot 与逻辑分区。此次检查仓库 tag 时观察到 `PKX110_16.0.0.212(CN01)_CN`、`PKX110_16.0.1.301(CN01)_CN`、`CPH2723_16.0.0.211(EX01)_IN` 和 `CPH2723_16.0.1.303(EX01)_IN`；这只是检查时的归档版本快照，不代表覆盖当前 `.501` 或所有 OxygenOS 版本。

## Root、完整性与隐藏相关线索

以下内容来自用户提供的 DerpFest 社区文本，统一标记为 `COMMUNITY_CLAIM + NOT_REVIEWED + NOT_TESTED`：

- 该文本称其 OnePlus 13T DerpFest 构建已内置 KernelSU-Next 与 SUSFS，并建议通过对应 KSU 模块隐藏部分 AOSP/Lineage 特征。
- 基础组合提及 Zygisk 与 LSPosed；出于实现透明性考虑，另提到 NeoZygisk 与 Vector。
- 针对解锁后 TEE/完整性问题，文本列出 TEESimulator-RS、TA-enhanced、PlayIntegrityFix-inject-s 与 `keybox.xml`。
- HMA-OSS 被列为隐藏可疑应用的候选工具，可导入现成预设。
- 文本观察到：root 环境下启用 Scene 的无障碍服务可能触发部分应用检测；出现问题时可把关闭该无障碍服务作为排查变量。

配套文件仅登记原始外链：[123 云盘打包链接（密码 `INT8`）](https://1817114020.share.123pan.cn/123pan/8MYVVv-QFmDh?pwd=INT8)。本仓库没有下载、检查或重新上传该包，也没有确认其中各文件的项目来源、版本、许可证、哈希或安全性。

这组资料只用于研究和兼容性排障。完整性绕过可能降低设备安全性、违反应用或服务规则；不得上传、共享或复用来源不明的私有 keybox。收录工具名称不代表本仓库推荐规避金融、支付、企业管理或反作弊系统的安全控制。

## Recovery 与 ROM 文件入口

| 参考 | 计划用途 | 当前状态 | 边界 |
|---|---|---|---|
| [kmiit/twrp_device_oplus_sm87xx](https://github.com/kmiit/twrp_device_oplus_sm87xx) | TWRP 设备树和 SM8750 recovery 实现参考 | `ALREADY_REFERENCED` | 已用于 [TWRP 研究文档](twrp.md)；新提交不增加实机结论 |
| [OnePlus pagani files on SourceForge](https://sourceforge.net/projects/oneplus-pagani/files/) | 国外 OnePlus 13T 类原生 ROM/文件发布入口 | `LINK_RECORDED + NOT_REVIEWED + NOT_TESTED` | 这是文件目录，不是兼容性或安全背书；下载前逐项确认维护者、版本、校验值和安装说明 |

## 后续采用顺序

1. 先固定候选仓库的 branch、commit 与许可证。
2. 与本仓库冻结的 LineageOS 23.2 manifest 做最小 diff，按“指纹、显示、相机、Wi-Fi、构建补丁”拆分。
3. 只在可回滚的测试分支构建；主机验证和设备验证分别记录。
4. 指纹与 HDR 必须补齐日志、触发条件和回归矩阵，不能仅以界面出现或单次成功判定。
5. 外部 ROM、recovery、root/隐藏模块和网盘包在完成来源与哈希审计前，不进入仓库 Release，也不提升为已验证方案。
