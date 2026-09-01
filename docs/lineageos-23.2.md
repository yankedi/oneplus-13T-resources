# LineageOS 23.2：构建、安装与硬件状态

更新时间：2026-09-01

2026-08-25 的本地源码、dirty 状态、active 输出混合快照和重新验证结果见[本地构建审计](local-build-audit-2026-08-25.md)。本页的 2026-08-14 完整 OTA 和 2026-08-16 设备状态结论保持原时间截面，不被未重新打包的增量输出覆盖。

2026-08-26 已通过 EDL 将连贯的 2026-08-14 Lineage recovery 写入 `recovery_b`，完整回读及 AVB 校验通过；Android B 和 Lineage Recovery 启动、root ADB 已确认。未改动其他分区，Recovery 全功能待验收。见 [ARB1 Linux EDL 实机报告](edl-arb1-device-validation-2026-08-26.md)。

2026-08-29 的同源构建已完成指纹相关 B 槽分区部署；完整录入、重启后亮屏认证、支付宝指纹登录/支付和 Google Play 支付验证通过。实现、提交、产物哈希和写入边界见[指纹修复与实机验证](fingerprint-fix-2026-08-30.md)。

2026-09-01 已从 `.501` 的 OPlus telephony framework 确认中国 ROM 的 NR 等级算法，并在 MCC 460 CarrierConfig 中固化。完整构建后仅向 slot B 写入 `product_b` 和 `vbmeta_system_b`；两个订阅连续 60 轮均从 level 2 修正为 level 4。见[中国 NR 信号等级修复与实机验证](china-nr-signal-fix-2026-09-01.md)。

## 构建产物

2026-08-14 的 `lineage_pagani-userdebug` 构建完成，产物为非官方、`userdebug/test-keys`：

| 字段 | 值 |
|---|---|
| 文件 | `lineage-23.2-20260814-UNOFFICIAL-pagani.zip` |
| 大小 | 2,503,659,880 bytes |
| SHA256 | `bcdec8cbcf5b42f42ff2e998ecb8f49bab6316fdc5d36d426725f0ee12bdde8e` |
| Android | 16 / SDK 36 |
| Kernel | 6.6.142 |
| OTA 类型 | A/B payload OTA |

主机侧检查：

- whole-package signature：PASS；
- payload signature、hash 和 size：PASS；
- AVB chain 与 hashtree：PASS（test key，root vbmeta flags `3`）；
- target-files VINTF：`compatible`；
- dynamic partition image-fit：`6,334,275,584 <= 13,329,498,112`。

这些结果是 `HOST_VERIFIED`，不能单独证明真机可启动或 userdata 安全。

## 冻结源码版本

首次成功构建/安装所用关键源码：

| Project | Commit |
|---|---|
| `OnePlus-13T-Development/android_device_oneplus_pagani` | `dfe9aa41e8154fc89dc217efae564bac6c376216` |
| `LineageOS/android_device_oneplus_sm8750-common` | `44ad18fa12b51983a36fb7ec67c54a6b4c032859` |
| `LineageOS/android_hardware_oplus` | `ec3b8211676f55ed09905c4c336356acedc040d3` |
| `LineageOS/android_kernel_oneplus_sm8750` | `999b95d4792dd41fba72a7914abe897cc3ee2ecd` |
| `LineageOS/android_kernel_oneplus_sm8750-devicetrees` | `ebb25e3526ad84cc5a1090a5a9242f33ff087bf2` |
| `LineageOS/android_kernel_oneplus_sm8750-modules` | `eab0a34a8b657ee22839c9b3878c3b6e687675f9` |

构建中的 pagani `proprietary-files.txt` 有最小本地修正，用于解决 AON/camera 重复 prebuilt 和跨分区模块冲突。由 `.501/.401` 生成的 proprietary vendor tree 含受限制的专有文件，不应公开分发。

## 安装时间线与关键教训

### 第一次写入 slot A

45 个 payload partitions（7 dynamic、8 boot-critical、30 firmware）成功写入目标 slot A，update_engine 返回成功。首次启动进入 Lineage Recovery，并报告 `init_user0_failed`。

决定性日志是 userdata 的 F2FS superblock 读取为无效数据。旧 ColorOS 的 metadata-encryption 链不能由该 Lineage 构建直接解开，因此不存在保留原 `/data` 的无损启动路径。此时没有擅自 wipe。

### 第二轮写入 slot B

在用户授权清除 userdata 后，第一次从 Lineage Recovery factory-reset 重启携带 `reboot,factory_reset`，OPlus ABL 回到健康的 stock slot A，并把 B 标成 unbootable；snapshot 被取消。该行为最初只是一项机制假设。

重新 sideload 后，直接尝试启动 B 仍因 TWRP 创建的 userdata 布局触发 `init_user0_failed`。随后在 **Lineage Recovery B 内执行 factory reset**，再经 bootloader 重启，Lineage 成功进入完整 userspace，并把 slot B 标记为 successful。

结论：

```text
当前 Lineage 构建的 /data 必须由 Lineage Recovery 创建。
不要使用 TWRP Format Data 为它准备 userdata。
```

## 最终成功状态（2026-08-16）

| 字段 | 值 |
|---|---|
| Booted slot | `_b` |
| Lineage | `23.2-20260814-UNOFFICIAL-pagani` |
| Android | 16 / SDK 36 |
| Kernel | `6.6.142-4k-g999b95d4792d` |
| `/data` | Lineage Recovery 新建，成功挂载 |
| Slot A | ColorOS `.501`，successful=yes |
| Slot B | Lineage，successful=yes |
| Snapshot state | none（已完成/合并） |

## 硬件冒烟测试

| 子系统 | 结果 | 边界 |
|---|---|---|
| Cellular RF / Mobile Data | PASS | NR SA 与数据连接稳定 |
| IMS / VoLTE / VoNR | PARTIAL→通话记录为正常 | 仍建议补充标准化长通话测试 |
| Wi-Fi | PASS | 当时 L2 双向约 500 Mbps |
| Bluetooth | PARTIAL | HAL/service 在；配对未按测试表复测 |
| NFC | PASS | enabled |
| GNSS | PARTIAL | service 注册；实际 fix 仍需标准化复测 |
| Camera | PASS | provider 工作，枚举 4 个设备；仍需补实拍矩阵 |
| Audio | PASS | QTI HAL、speaker patch、低延迟流工作 |
| Sensors | PASS | accel/gyro、mag、light/prox 注册 |
| USB / ADB | PASS | 稳定 |
| Charging | PASS | 基础充电状态正常 |
| Display | PASS | 亮度与唤醒正常，UDFPS overlay 可显示 |
| Fingerprint enrollment | FAIL (`SUPERSEDED`) | 2026-08-16 时间截面为 `waitUiready` 超时和 `15001`；已由 2026-08-29 修复验证取代 |

这里的 Camera `PASS` 只表示当前 AOSP/Lineage Camera provider、设备枚举和基础拍摄链工作，不表示 OPlus/ColorOS 原厂相机已经完成移植。关于 ColorOS ODM 文件和 OPlus framework 依赖的新增线索见 [OPlus / ColorOS 相机移植线索](camera-porting-clue.md)。

### 2.4 GHz Wi-Fi 的验证边界

上表中的 Wi-Fi `PASS` 没有单独记录频段，因此不能外推为 2.4 GHz 已完成专项验证。[`zzkeier/android_device_oneplus_sm8750-common@9280681`](https://github.com/zzkeier/android_device_oneplus_sm8750-common/commit/92806812c82f10d42ea663c1b6348c2a97294d7b) 是一项待参考的 pagani 2.4 GHz 候选修复：它扩展 ueventd firmware 搜索路径，并把两个额外的 ODM Wi-Fi 配置文件加入 proprietary 列表。

该差异不在本次成功构建冻结的 `LineageOS/android_device_oneplus_sm8750-common@44ad18f` 中，尚未合入、构建或实机测试。具体文件变化和采用前检查项见[候选参考目录](pending-references.md#24-ghz-wi-fi--common-tree-候选修复)。

## UDFPS 修复结论

原故障的 `waitUiready` 约 502 ms 超时和 vendor error `15001` 已解决。最终没有替换指纹 APK、HAL、TA、firmware 或整棵 `hardware/oplus`，也没有屏蔽错误码。

真实触摸 down/up 现在由 SM8750 OPlus 显示内核模块写入 `notify_fppress=1/0`；pagani 启用 QTI OPlus UDFPS，common 设备树设置精确节点权限，SystemUI 只在 finger-up、取消和 overlay 关闭路径补写 `0`。该顺序既让 HAL 及时收到 UI_READY，也清除了认证成功后可能缺失物理 finger-up 所造成的持续高亮。

完整源码和证据见[2026-08-30 指纹修复报告](fingerprint-fix-2026-08-30.md)。AOD/熄屏认证仍为 `NOT_TESTED`。

## 蜂窝信号格

刷入前 60 秒采样为 NR SA，两个订阅平滑 SSRSRP 均约 `-90 dBm`，数据正常，但 framework 连续 60 轮均报告 level 2；运行时采用 AOSP 默认 NR SSRSRP 门限 `[-110, -90, -80, -65]`。

早期 CarrierConfig APK 审计没有在中国运营商块中找到 NR 门限。继续提取 `.501` 的 `oplus-telephony-common-ext.jar` 后确认，`OplusSignalStrengthStandard.getNrLevel()` 直接复用 LTE 等级算法，其默认边界为 `-126,-121,-114,-105,-44`。AOSP 只接受四个等级边界，`-44` 是有效上限，因此 MCC 460 配置为 `[-126, -121, -114, -105]`。

完整构建并部署 `product_b`、`vbmeta_system_b` 后，两个订阅的有效 CarrierConfig 均为新门限，连续 60 轮 framework level 均为 4，驻网和数据连接保持正常。修复提交为 [`42e94f0`](https://github.com/yankedi/android_device_oneplus-13T_common/commit/42e94f0e61f63c9622a6bfc90878098da682f7f1)。该结果只证明显示等级映射修复，不表示射频性能提升；完整证据和适用边界见[信号修复报告](china-nr-signal-fix-2026-09-01.md)。

## 不应直接移植的内容

- `.500` generation 的 ABL/XBL/TZ/HYP/AOP/modem/DSP；
- 完整 vendor tree 或 camera blobs；
- 旧的 Fppress/SensorProps 重复实现；
- 未复现的 vendor error `10108` 抑制；
- aggressive power hints、LTPO magic write 或整批 framework patch。

详细差异见 [AOSP 源码差异审计摘要](aosp-gap-audit.md)。
