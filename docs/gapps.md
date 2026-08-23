# GApps / NikGapps 事件记录

更新时间：2026-08-23

## 范围

本页记录 2026-08-23 在 OnePlus 13T、LineageOS 23.2、Android 16 上发生的 NikGapps 调试事件。原始日志和专有 APK 不公开；这里只保存足以复核的脱敏事实。

## 1. Google Clock 导致 bootloop

`omni-debug` 中保存了连续的 `system_server_crash`。从 12:01:30 起，每约 5 秒重复出现同一异常：

```text
java.lang.IllegalStateException:
Signature|privileged permissions not in privileged permission allowlist
```

触发包：

```text
com.android.deskclock
/product/priv-app/PrebuiltDeskClockGoogle_76007351
```

缺少的两项权限：

```text
android.permission.START_FOREGROUND_SERVICES_FROM_BACKGROUND
android.permission.CONTROL_DISPLAY_COLOR_TRANSFORMS
```

AOSP 在 `ro.control_privapp_permissions=enforce` 下要求 privileged app 请求的相关权限出现在 allowlist 中；未列出时可以拒绝完成系统启动。因此这里不是普通 DeskClock 应用崩溃，而是 Package/Permission systemReady 阶段终止 `system_server`。

状态：`DEVICE_VERIFIED`。

## 2. 生成过的修复 ZIP

曾生成 `NikGapps-GoogleClock-Android16-PrivappFix.zip`，设计为在 `/product/etc/permissions/` 增加针对 `com.android.deskclock` 的两项 allowlist。文件内容与目标问题一致，但现有证据没有证明它已经成功刷入并完成一次正常启动。

状态：

```text
ARTIFACT_CREATED / NOT_TESTED
```

不要把“ZIP 已生成”写成“bootloop 已修复”。长期方案应把正确 allowlist 纳入可重现的 GApps/ROM 构建，并重新执行干净安装和首次启动验证。

## 3. `UnInstall.zip` 实际执行了卸载

Recovery 日志确认，原文件被重命名：

```text
NikGapps-omni-arm64-16-20260222-signed.zip -> UnInstall.zip
```

随后 NikGapps 把模式切换为 `uninstall_by_name`。即使日志中保存的 `nikgapps.config` 仍写着 `Mode=install`，官方 NikGapps 机制也规定：文件名为 `UnInstall.zip` 时按卸载器运行，不依赖该 config 字段。

现场 package log 确认被删除的组件包括：

- Google Play Store、GmsCore、Google Services Framework；
- Setup Wizard、Google Restore、One Time Initializer；
- Google Dialer、Google Clock、Messages、Contacts；
- Maps、Photos、Calendar、Drive、Keep、GBoard；
- Carrier Services、Device Health Services 等。

Google Dialer 的 APK、overlay、support JAR 与 permissions XML 均被移除；Google Clock 的 APK、overlay 与 permissions XML 也被移除。因此之后出现“电话/时钟不见了”与这次卸载记录直接一致，不应误判为 Launcher 隐藏或单个 APK 损坏。

状态：`DEVICE_VERIFIED`。

## 4. 临时 priv-app log 模式

曾生成 `KernelSU-Privapp-LogMode-Temporary.zip`，目标是临时设置：

```text
ro.control_privapp_permissions=log
```

它只适合收集缺失 allowlist 项，不能作为永久修复。现有材料没有确认模块是否成功启用、重启后 property 值或最终测试结果。

状态：

```text
ARTIFACT_CREATED / ACTIVATION_NOT_CONFIRMED
```

## 5. 另一个独立的 Settings crash

13:04 左右还记录到 `com.android.settings` 在保存锁屏设置后发生 `NullPointerException`，调用链位于 `ScreenLockPreferenceDetailsUtils` / `LockScreenSafetySource`。它发生在系统已经运行后，堆栈与 Google Clock priv-app allowlist bootloop 不同；目前没有足够证据把二者归为同一根因。

状态：`DEVICE_VERIFIED / ROOT_CAUSE_UNKNOWN`。

## 公开来源

- [NikGapps 项目](https://github.com/nikgapps/project)
- [NikGapps 官方卸载说明](https://nikgapps.com/misc/2023/03/04/Uninstall-NikGapps.html)
- [NikGapps 配置说明](https://nikgapps.com/misc/2022/02/22/NikGapps-Config.html)
- [AOSP privileged permission allowlist](https://source.android.com/docs/core/permissions/perms-allowlist)

## 下一次验证建议

1. 从干净 ROM/槽位开始，先记录 system/product/system_ext 剩余空间。
2. 使用原始、未改名且哈希已确认的 Android 16 GApps 包。
3. 安装前检查每个 `/product/priv-app` 的 requested privileged permissions 与同分区 allowlist。
4. 首次启动前保留 recovery 日志；首次启动后检查 `system_server_crash`、PackageManager 和 permissions 状态。
5. 只有在完整启动、Dialer/Clock 可用、通话/闹钟工作后，才能把状态改为 `DEVICE_VERIFIED`。

