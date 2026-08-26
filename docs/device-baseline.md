# 设备与版本基线

更新时间：2026-08-26

## 设备身份

| 字段 | 值 | 状态 |
|---|---|---|
| 市场名称 | OnePlus 13T | `DEVICE_VERIFIED` |
| 型号 | `PKX110` | `DEVICE_VERIFIED` |
| Product device | `OP60F5L1` | `DEVICE_VERIFIED` |
| AOSP 代号 | `pagani` | `SOURCE_VERIFIED` + `DEVICE_VERIFIED` |
| SoC 平台 | Qualcomm SM8750，平台名 `sun` | `DEVICE_VERIFIED` |
| OPlus project | `24821` | `DEVICE_VERIFIED` |
| 屏幕 | 1216 × 2640 | `DEVICE_VERIFIED` |

## 状态时间线

### 2026-08-14：ColorOS `.302`

- ROM：`PKX110_15.0.2.302(CN01)` A.46，Android 15，安全补丁 2025-06-01。
- Kernel：6.6.66 Android 15 GKI。
- 活动槽：`_b`。
- bootloader 后续通过 bootloader fastboot 直接确认 `unlocked=yes`；早期 Android property 中出现过 `locked/green` 与 bootconfig `unlocked/orange` 冲突，因此旧的“可能锁定”判断已被后续直接证据取代。
- KernelSU 只修改了 `init_boot_b`：stock `init` 被 wrapper 替代，原始 `init` 保存在 `init.real`，并增加 `kernelsu.ko`。当时比较的 `boot_b`、`vendor_boot_b`、`dtbo_b`、`recovery_b` 与 vbmeta 系列仍符合 stock 分类。

### 2026-08-15：TWRP 与完整备份

- `recovery_b` 写入 TWRP 3.7.1_16-OnePlus_13_T，并在原厂 recovery 启动路径中完成非破坏性验收。
- 对 boot-critical、firmware、NV/calibration、super、userdata、metadata 和用户数据建立分层备份；276/276 个文件通过 SHA256 清单验证。
- 备份过程未执行 wipe、format、slot 修改或 ROM 安装。

### 2026-08-16：LineageOS 23.2

- 非官方 LineageOS 23.2 Android 16 OTA 先后经过 slot A 与 slot B 安装实验。
- 最终 slot B 成功进入完整 Android userspace，并被标记为 `successful=yes`。
- slot A 保留为可启动的 ColorOS `16.0.3.501(CN01)` 救援槽。
- `recovery_a` 为经验证的 TWRP，`recovery_b` 为 Lineage Recovery。
- Lineage `/data` 必须由 Lineage Recovery 执行 factory reset 创建；TWRP 格式化出的 metadata-encrypted F2FS 在该 Lineage 构建上会触发 `init_user0_failed`。

### 2026-08-23：GApps 调试

- 设备仍报告 LineageOS Android 16、kernel 6.6.142；KernelSU 与 ZygiskNext 已启动。
- NikGapps Google Clock 的 priv-app allowlist 缺失导致 `system_server` 循环崩溃。
- 后续执行的 `UnInstall.zip` 流程删除了 Google Dialer、Clock、Play Store、GMS 等组件。详见 [GApps 事件记录](gapps.md)。

### 2026-08-25/26：ARB1 EDL 与 Recovery 更新

- Android root 与 EDL 读取的 `xbl_config_b` 哈希一致，解析为 OEM metadata `3.0`、ARB `1`；硬件计数器未测。
- Linux 下免在线授权进入 Firehose；备份原 TWRP 后，将 `recovery_b` 更换为 Lineage Recovery，100 MiB 完整回读、AVB 与非目标启动分区检查通过。
- EDL 普通重启确认 Android B 启动完成，再经 ADB 启动 `23.2-20260814-UNOFFICIAL-pagani` Lineage Recovery，root ADB 可用。

A 槽未改动或重测；Recovery 全功能待验收。配置和注意事项见 [实机报告](edl-arb1-device-validation-2026-08-26.md)。

## 已验证的 Stock `.302` 资产

| 资产 | SHA256 | 说明 |
|---|---|---|
| Full OTA | `f15dd4f7db6451c0779772c3a9c089fd8fa1de9696d4576e991bb7cd6d2224b0` | 包签名与 payload 签名通过 |
| Payload | `3b7fba50249448d99d684f47aa0f3df9c5cedfe4d27153d8aab71cd24a5a4855` | 与 OTA metadata 一致 |
| `boot.img` | `edc5352f0ec17338221d4af36f34ab0857ae8eb7a78201ca2faafa50330300ef` | stock |
| `init_boot.img` | `8c32309ba9cb6cd1e5243bd8a3a537264f2de80526f1ac3c5993b3845994d00f` | clean stock；不同于 KernelSU 运行态 |
| `vendor_boot.img` | `e12302d6cf4ed62e0740855a5314a73596741be0824f84ed6cc9ce1aa6d1b7f7` | stock |
| `dtbo.img` | `4e110c85affa910d542a98e8dd6591bd0fb0f8068c986697cd7083c4b1335206` | stock |
| `recovery.img` | `86a343cb3907a21002de7ad013a1054e850a947382892dd4a6227f9afe683cb0` | header v4，kernelless |

哈希只用于识别已经验证过的文件，不提供下载，也不能证明其他来源的同名文件安全。

## 公开 Stock `.501` 参照包

2026-08-24 下载并审计了与救援槽版本相同的公开完整 OTA。它不是当前 A 槽的实时 dump，因此只建立“公开发行包与冻结源码”的主机侧参照：

| 资产 | 大小 | SHA-256 | 状态 |
|---|---:|---|---|
| `PKX110_16.0.3.501(CN01)` Full OTA | 8,430,217,707 bytes | `5df62c0fb98a540950c69667ac9abfff8635fc502e8cb123af7e0f1175122844` | ZIP CRC `PASS` |
| `payload.bin` | 8,430,211,175 bytes | `9fdfc47e2f5a147d68f074e2fb2602a575eb5926d60d7a230892c01a2b40c88c` | 独立流式哈希与 `FILE_HASH` 一致 |
| 选定的 11 个分区镜像 | 见交叉审计 | 各自与 payload manifest partition hash 一致 | `HOST_VERIFIED` |

OTA 和提取镜像不进入 Git；仓库只记录哈希、路径、数量、冻结源码和结论。完整分区表及指纹、Wi-Fi、相机、运营商交叉结果见 [`.501` 原厂包与现有设备树交叉审计](stock-501-cross-audit.md)。2026-08-24 的这次公开 OTA 提取没有独立复算 ARB；后续已安装 `xbl_config_b` 的实读是另一项证据，见下文。

## Firmware 代际边界

| 标识 | 用途 | 结论 |
|---|---|---|
| `.302` | 最初实机与完整备份基线 | 已验证 |
| `.501` | pagani Android 16 proprietary/firmware 与救援槽基线 | 已安装并启动 |
| `.401` | OnePlus 13 SM8750 common blob 基线 | 用于构建，不是 PKX110 firmware 包 |
| `.500` | Oneplus13T-AOSP 历史 reference generation | 仅用于源码比较，未安装；不得代替 `.501` |

源码差异、信号格或 UDFPS UI 问题均不能作为盲目更换 ABL/XBL/TZ/HYP/AOP/modem/DSP 的理由。

## `.501` 的 ARB 边界

公开的 OnePlus ARB 跟踪数据在 2026-08-23 将 `PKX110_16.0.3.501(CN01)` 标为 rollback index `1`，而 `PKX110_16.0.2.400(CN01)` 标为 `0`，作为版本边界的 `SOURCE_VERIFIED` 交叉证据保留。

2026-08-25/26 已独立读取本机 `xbl_config_b` 并解析出 ARB `1`：完整 409600 字节 SHA-256 为 `a29081b7eb20752dee63efd21f6f984531e4560c153a09d115aab644736eda28`，有效内容与冻结 `.501` 镜像一致。证据为 `DEVICE_VERIFIED + HOST_VERIFIED`，但只证明已安装镜像声明值，熔丝/RPMB 计数器仍为 `NOT_TESTED`。

ARB 约束的是设备是否接受更低 rollback index 的启动组件，并不等同于“9008 接口消失”或“任何 Firehose 都不可加载”。相关概念、社区时间线和已实测范围见 [EDL、Firehose 与 ARB1](edl-arb.md)。

## 历史槽位布局与最近更新

以下表格保持 2026-08-16 的时间截面：

| 槽位 | 系统 | Recovery | 健康状态（2026-08-16） |
|---|---|---|---|
| A | ColorOS `16.0.3.501(CN01)` | TWRP 3.7.1_16 | successful=yes，unbootable=no |
| B | LineageOS 23.2 unofficial | Lineage Recovery | successful=yes，unbootable=no |

2026-08-26 最近更新：本次操作前 B 槽 Recovery 实读为 TWRP，现已通过 EDL 更换为 Lineage Recovery 并实际启动；普通 Android B 启动也已确认。A 槽内容与 `successful/unbootable` 标志没有重新读取，不更新其历史健康结论。

槽位状态会随 OTA、恢复、刷写和 bootloader 行为变化；任何后续操作前必须重新读取，不能永久依赖这张表。
