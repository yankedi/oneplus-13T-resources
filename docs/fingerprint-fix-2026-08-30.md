# OnePlus 13T LineageOS 指纹修复与实机验证

更新时间：2026-09-01（验证发生于 2026-08-29/30）

## 结论

在 `PKX110` / `pagani`、LineageOS 23.2 / Android 16、slot B 上，屏下指纹的完整录入、亮屏认证和第三方支付授权已经实机验证。最终方案没有替换指纹 HAL、TA、固件或整棵 `hardware/oplus`，也没有屏蔽 vendor 错误码。

最终事件路径为：

```text
真实触摸事件
  -> SM8750 OPlus 显示内核模块写入 notify_fppress=1/0
  -> QTI OPlus UDFPS / UI_READY / 指纹 HAL 采图
  -> SystemUI 只在 finger-up、取消或 overlay 关闭时补写 0
```

内核是真实按下状态的唯一 `1` writer；SystemUI 只负责幂等清理 `0`，用于覆盖认证成功后触摸上报先关闭、物理 finger-up 不再到达显示 notifier 的情况。这一顺序修复了录入链路，也避免了解锁后指纹区域持续高亮。

## 固化源码

LineageOS 是由 `repo` manifest 组合的多 Git 仓库源码树。本次修复跨越 Framework、pagani 设备配置、SM8750 公共配置和内核模块，因此分别固化到四个 Fork：

| 层 | Fork | 上游 | 修复提交 | `lineage-23.2` HEAD |
|---|---|---|---|---|
| SystemUI | [`yankedi/oneplus-13T_android_frameworks_base`](https://github.com/yankedi/oneplus-13T_android_frameworks_base) | [`LineageOS/android_frameworks_base`](https://github.com/LineageOS/android_frameworks_base) | `ed26e45fde9f` | `b788900f949f2186de40c44c53492285e14a198c` |
| pagani 设备树 | [`yankedi/android_device_oneplus_pagani`](https://github.com/yankedi/android_device_oneplus_pagani) | [`OnePlus-13T-Development/android_device_oneplus_pagani`](https://github.com/OnePlus-13T-Development/android_device_oneplus_pagani) | `877593f`, `2e389d0`, `4d2cd9e` | `4d2cd9eb9ca7f1a19cc7d89c4722ba74d564152d` |
| SM8750 公共设备树 | [`yankedi/android_device_oneplus-13T_common`](https://github.com/yankedi/android_device_oneplus-13T_common) | [`LineageOS/android_device_oneplus_sm8750-common`](https://github.com/LineageOS/android_device_oneplus_sm8750-common) | `62cf962` | `42e94f0e61f63c9622a6bfc90878098da682f7f1` |
| SM8750 内核模块 | [`yankedi/android_kernel_oneplus-13T_sm8750-modules`](https://github.com/yankedi/android_kernel_oneplus-13T_sm8750-modules) | [`LineageOS/android_kernel_oneplus_sm8750-modules`](https://github.com/LineageOS/android_kernel_oneplus_sm8750-modules) | `5b64073` | `a9bb81bd2819a23ae56f3253981bae129313395f` |

四个 Fork 统一以 `lineage-23.2` 作为构建和提交分支。指纹提交均已包含在该分支；历史 `pagani-fingerprint-fix` 只是合并前的临时指针，删除它不会删除已合并的提交，也不是构建依赖。合并没有使用强推。SM8750 公共设备树主分支随后增加了已验证的中国 NR 信号等级修复 `42e94f0`。

### 各提交职责

- `ed26e45fde9f`：SystemUI 在 finger-up、overlay 隐藏等清理路径写入 `notify_fppress=0`，不负责写入 `1`。
- `877593fbad0`：为 pagani 启用 QTI display 的 OPlus UDFPS 编译路径。
- `2e389d031287`：为 `notify_fppress` 建立专用 SELinux 类型，只允许 `platform_app` 搜索父目录并对该专用节点执行 `open/getattr/write`。
- `4d2cd9eb9ca`：使用源码树内固定的 `depmod`，消除宿主机工具差异。
- `62cf9629ebbb`：通过 ueventd 将 `notify_fppress` 设置为 `0664 system:system`。
- `5b64073d036e`：让真实触摸 down/up 调用 OPlus OFP press 更新，并调整 notifier 优先级，使指纹监听先收到触摸事件、显示侧随后产生 UI_READY。

最终没有移植候选 `OplusFodShim.cpp`。设备没有候选实现依赖的 `fp_state`，而内核真实触摸路径能够提供更直接的事件源；现有 `SensorPropsShim` 保持不变。

## 合并时的上游差异

合并前，`pagani-fingerprint-fix` 与对应上游 `lineage-23.2` 的关系如下：

| 仓库 | 功能分支领先 | 功能分支落后 | 落后的上游提交 |
|---|---:|---:|---|
| frameworks/base | 1 | 3 | `094652839e07`, `480e2fbe83e3`, `68ee585f54bc` |
| device/oneplus/pagani | 3 | 0 | 无 |
| device/oneplus/sm8750-common | 1 | 0 | 无 |
| kernel/oneplus/sm8750-modules | 1 | 1 | `3613bfa7517a` |

Framework 的三个上游提交依次修复 TV 全局操作焦点、允许 `/data` 中的 microG location provider、在 NLP 不可用时使用 GPS。内核上游提交同步无线充电与 MMI charging 状态。它们已与指纹修复一并保留在 Fork 的 `lineage-23.2` 中。

## 构建与部署

完整同源构建产物：

| 字段 | 值 |
|---|---|
| 文件 | `lineage-23.2-20260829-UNOFFICIAL-pagani.zip` |
| 大小 | `2,503,305,953` bytes |
| SHA-256 | `33bd6558ff19437f7436dde186623a617d7dcad971a4cdf62a141cd258a6f9c8` |
| whole-package / payload 签名与 payload hash/size | PASS |
| AVB chain / VINTF / dynamic partition capacity | PASS |

当时设备运行 slot B。标准 A/B OTA 会写入 inactive slot A，因此没有执行标准 OTA；只把同源产物中的 `system_ext_b`、`vendor_b`、`odm_b`、`vendor_dlkm_b`、`vbmeta_system_b` 和 `vbmeta_vendor_b` 写入 B。没有命令指向 slot A，也没有改动 boot、init_boot、vendor_boot、firmware、userdata 或 super metadata。

写入前保存了完整 `super` 回滚镜像，大小 `14,956,888,064` bytes，SHA-256 为 `eb7e7b95e9681f3f8437e79d4f76ec07104e2d0adf6b44ca3f12f62f5c8324c1`，并保存匹配的 B 槽 vbmeta 镜像。备份本身不进入本仓库。

## 实机验证

- 从空模板开始完成一枚指纹录入，剩余次数从 `19` 连续下降到 `0`。
- 录入阶段 HBM on/off 为 `24/24`，最终 HBM 和 UI_READY 均回到 `0`。
- 重启前完成三次强生物识别认证；重启后模板仍存在，并再次完成两次认证。
- 未观察到 `waitUiready -> 15001`、相关 `sysfs_notify_fppress` AVC、SystemUI/指纹 HAL fatal 或 tombstone。
- 没有再次出现指纹区域持续高亮。
- 设备持有者已实机启用支付宝指纹登录并完成指纹支付。
- 设备持有者已实机使用指纹完成 Google Play 支付验证。

因此，指纹录入、亮屏认证、支付宝指纹登录/支付和 Google Play 支付验证标记为 `DEVICE_VERIFIED`。

## 边界

- AOD/熄屏认证仍为 `NOT_TESTED`。
- 支付验证依据设备持有者观察到的应用界面结果；本轮没有采集支付应用的诊断日志。
- 支付验证证明当前实机的 Android BiometricPrompt、强生物识别和相关支付调用链可用，不保证所有第三方应用都会忽略解锁 bootloader、KernelSU、自定义 ROM、test-keys 或设备完整性策略。
- 实机验证对应 2026-08-29 的构建基线。合并后的 Fork 主线另外包含上节列出的上游提交，尚未重新完整构建和刷机验证。
