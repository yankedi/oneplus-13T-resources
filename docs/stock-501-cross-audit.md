# `.501` 原厂包与现有设备树交叉审计

最后核对：2026-09-01（原始交叉审计：2026-08-24；信号 framework 补充：2026-09-01）

本页把当前已成功启动的 OnePlus 13T LineageOS 23.2 冻结源码、已筛选的设备树／`hardware/oplus` 候选，以及与 slot A 同版本的 `PKX110_16.0.3.501(CN01)` 公开完整 OTA 放到同一证据链中。目标是缩小现有问题的源码范围，不是从原厂包拼装可刷写镜像，也不是证明任何候选已经修复实机。

## 结论先行

1. **保留当前已构建基线。** 没有一套候选树同时具备更接近当前源码、完整依赖和 PKX110 实机验证，不能整棵替换 pagani、common 或 `hardware/oplus`。
2. **指纹仍是 P0，但现有节点访问基础比预想完整。** 现有日志把故障限定在 UFF HAL 的 UI-ready／`notify_fppress` 门：等待约 502 ms 后返回 vendor error `15001`。`.501` 的 display init 脚本会把该节点设为 `system:system 0666`；冻结 Lineage common 会导入这份脚本，`hardware/oplus` 也已有 fingerprint HAL 对 `vendor_sysfs_graphics` 的读写规则。第一轮候选应只移植 RandomLemon 的事件桥并验证现有访问链，只有实机出现权限错误时才补规则。
3. **2.4 GHz Wi-Fi 是待复测候选，不是已确认故障。** `.501` 中三个 WCNSS 配置均存在，但正常启动脚本默认仍选择基础配置；zzkeier 提交加入额外文件和 firmware 搜索路径，却没有把普通运行路径明确切到它们。现有 Wi-Fi `PASS` 又没有记录频段，因此必须分频段验证实际加载文件。
4. **相机清单与 `.501` 对齐，不是当前优先替换项。** 当前构建已枚举四个 camera device 且基础拍摄通过；kinginu `.501` 标签下 101 个 ODM 文件与本次原厂包逐文件哈希一致，冻结 pagani 清单的 916 个相机相关路径也全部存在。它们证明来源与清单一致，不证明 ColorOS framework 功能已移植。
5. **后续 framework 审计已补全中国 NR 信号格答案。** 本页最初检查的 CarrierConfig APK 没有在中国运营商块列出 NR 表；2026-09-01 继续检查 `.501` 的 OPlus telephony framework 后确认，NR 复用默认 LTE 等级边界 `-126,-121,-114,-105,-44`。四个 AOSP 门限已固化并实机验证。Google Clock 事件仍是权限 allowlist／安装流程问题，与设备树或原厂 vendor 分开处理。

## 审计对象与证据边界

### 已构建冻结基线

| 路径 | 仓库与提交 | 状态 |
|---|---|---|
| `device/oneplus/pagani` | [`OnePlus-13T-Development@dfe9aa4`](https://github.com/OnePlus-13T-Development/android_device_oneplus_pagani/tree/dfe9aa41e8154fc89dc217efae564bac6c376216) | `DEVICE_VERIFIED` 构建输入 |
| `device/oneplus/sm8750-common` | [`LineageOS@44ad18f`](https://github.com/LineageOS/android_device_oneplus_sm8750-common/tree/44ad18fa12b51983a36fb7ec67c54a6b4c032859) | `DEVICE_VERIFIED` 构建输入 |
| `hardware/oplus` | [`LineageOS@ec3b821`](https://github.com/LineageOS/android_hardware_oplus/tree/ec3b8211676f55ed09905c4c336356acedc040d3) | `DEVICE_VERIFIED` 构建输入 |

“设备验证”只表示这一组源码进入了成功启动并完成既有冒烟测试的构建；指纹录入本身明确为 `FAIL`。本页不把后来未提交的本地实验改动写回冻结 commit。

### `.501` 参照包

本轮使用的是与 slot A 版本相同的公开完整 OTA，不是从用户当前 A 槽读取的实时分区副本：

| 字段 | 值 |
|---|---|
| 用户可见版本 | `PKX110_16.0.3.501(CN01)` |
| OTA build | `PKX110_11.C.39_1390_202601062328` |
| Android / SDK | Android 16 / 36 |
| 安全补丁 | 2026-01-01 |
| 文件大小 | 8,430,217,707 bytes |
| MD5 | `4b08985d35a0fa74c5041ac96d5efb0b` |
| SHA-256 | `5df62c0fb98a540950c69667ac9abfff8635fc502e8cb123af7e0f1175122844` |
| `payload.bin` 大小 | 8,430,211,175 bytes |
| `payload.bin` SHA-256 | `9fdfc47e2f5a147d68f074e2fb2602a575eb5926d60d7a230892c01a2b40c88c`（独立流式复算，与 `FILE_HASH` 解码值一致） |
| 本轮分段下载 | 约 34 分 43 秒；平均约 4.05 MB/s，不含重组和校验 |

公开 OTA 由 [Daniel Springer OTA 索引](https://roms.danielspringer.at/index.php?view=ota)解析到 OPlus CDN。分析只保留版本、大小、哈希、payload 分区元数据和必要的文件级结论；OTA、镜像、APK、ELF 和 proprietary blobs 均不进入 Git。

### 方法

- 用 Git object、merge base 和逐路径 diff 比较冻结源码；不以 README 或 commit 标题代替代码。
- 对完整 OTA 先复算整体哈希并执行 ZIP CRC 检查，再解析 payload manifest；manifest 共 54 个分区，对本页所需的 11 个提取镜像复算其 partition hash。
- 只读提取 `odm`、`vendor`、`vendor_boot`、`system_ext`、`product`、`my_product`、`my_carrier` 和 `my_region` 等相关分区。
- 只记录路径、依赖、配置键、数量与哈希；不提交专有内容。
- 本轮没有向手机写入任何分区，也没有把源码候选编译或刷入设备。因此所有修复候选继续标为 `NOT_APPLIED + NOT_TESTED`。

## 指纹：依赖链而不是单文件

### 当前故障边界

已有设备证据形成的最短链路是：

```text
SystemUI finger down/up
  -> OPlus display/OFP 事件桥
  -> /sys/kernel/oplus_display/notify_fppress
  -> UFF HAL waitUiready / postUiready
  -> enrollment frame
```

sensor、TA、UFF 服务、VINTF、overlay 和触摸按下均已出现；失败点是 HAL 没有在约 502 ms 窗口内看到期望的 UI-ready 条件，随后上报 `15001`。这使“换指纹 firmware”“换整套 vendor”“忽略 10108”都不符合当前证据。

### 候选实现对比

| 候选 | 实际源码状态 | 能说明什么 | 仍缺什么 |
|---|---|---|---|
| 当前 LineageOS | v3 SensorProps shim 已注入 UFF 服务；共享 shim 支持 `iconlocation`、DRM 分辨率和 radius；common 导入原厂 display rc，通用 SELinux 规则允许 fingerprint HAL 读写 `vendor_sysfs_graphics` | 传感器参数和节点访问基础已进入成功构建 | 没有 OplusFod 事件桥；仍需实机确认节点模式、上下文和 AVC 状态 |
| [`RandomLemon hardware/oplus@5e5ff9a`](https://github.com/RandomLemon/android_hardware_oplus/tree/5e5ff9a5bfce2183e538c4a00b3fd51d4e8719c8) | 把 `OplusFodShim.cpp` 编入 v2/v3/v4 shim；监测 `fp_state`，处理 pointer 与 acquired `22/23`，写 `notify_fppress=1/0` | 提供与当前超时边界直接相符的完整事件桥候选 | 实际服务注入／加载证明、AOD/取消/HBM 回归和设备测试；访问规则只在实机拒绝时补 |
| zzkeier pagani/common | pagani 有本地 `FppressShim`、策略；common 有 ueventd 权限 | 展示了 shim、节点权限、策略三部分的组合 | pagani `.pagani` shim 没有在所查 `device.mk`／blob fixup 中接线；common 仍注入 `.oplus` v3 |
| ABNOTF patches/tree | SystemUI 补丁在 finger down/up 直接写节点 | 另一种 framework 侧触发位置 | 构建指南只列相机补丁；SystemUI writer domain 与 HAL 不同，现有 HAL 规则不能自动证明它可写，补丁也未设备验证 |
| renhiyama common | 除 v3 SensorProps 外还注入 boot-state property shim；增加指纹属性与 touchDaemon 策略 | 可作为解锁状态／enrollment anti-tamper 的第二层候选 | 没有 `notify_fppress` 桥；源码注释不能证明它解释当前 `15001` |
| Neveark common | 注入 v3 SensorProps shim | 适合比较传感器参数路径 | 没有第二 enrollment shim，也没有事件桥 |

RandomLemon 与当前 LineageOS `hardware/oplus` 的 merge base 为 `dad4fa2064230191bb616f17cf660423dfd31d94`，两侧分别有 30／9 个独有提交，HEAD 间 1,786 个路径不同。限定到 `fingerprint/` 后仅有 `Android.bp`、`OplusFodShim.cpp` 和 `SensorPropsShim.cpp` 三个路径不同。因此合理的候选不是换树，而是保留当前 SensorProps，仅定向移植事件桥与必要依赖。

### 最小验证组

一次候选构建应把以下项目作为同一变更组，但在日志中分别观测：

1. 在当前 `hardware/oplus` 上移植 `OplusFodShim.cpp` 与明确的 `Android.bp` 依赖，不回退 SensorProps，也不整棵换树。
2. 第一轮先沿用现有 display rc 与通用 SELinux 访问链；在实机记录节点存在性、`ls -lZ`、实际 writer domain 和 AVC。只有出现真实拒绝时才补最小 ueventd／SELinux 规则。
3. 构建后用 ELF `NEEDED`、进程 maps 和 shim constructor 日志证明代码真的加载，不能只看源码文件存在。
4. 录入场景同时记录 finger down/up、节点值、`waitUiready`、`postUiready`、`onUiReady`、vendor code 和 HBM 状态。
5. 回归认证成功、认证失败、取消、息屏、AOD、快速抬手、导航手势和 session close，确认每条路径都会清理 `notify_fppress=0`。
6. 只有事件桥已加载且 `15001` 仍存在时，才单独试验 renhiyama 的 boot-state shim；不要第一批混入。

## `.501` 文件级交叉结果

### 包、payload 与分区完整性

完整包有 6 个 ZIP entry，`unzip -t` 的 CRC 检查通过，payload manifest 为版本 2、block size 4096。下面列出的镜像均由 payload 提取，SHA-256 与 manifest 的对应 partition hash 相同：

| 分区 | 大小（bytes） | SHA-256 |
|---|---:|---|
| `odm` | 1,871,388,672 | `2067dece06a5ec9654ac9a541df74171c33fc6c29e1b61cfb744101085bf97d6` |
| `my_product` | 1,625,628,672 | `bd06f48316b9ed45344b03ec18e024b194fc6ffe9ddcb06cce27ee861d8764c7` |
| `vendor` | 754,294,784 | `3727bc78f03fd8e25e2e2cbe230b88030d367a252b16576118ddf5bd60a19de3` |
| `system_ext` | 853,102,592 | `b7008bf941f7dbea63d5cb76029601fcf03b6ea8a76544f8fe546b2867cceac5` |
| `product` | 9,203,712 | `29296bc84711be82a25f535002652edb76fde0a7a348cf188877899a3286a167` |
| `my_carrier` | 335,872 | `407e750d408567cd2296c8507faea14b342c9bea44a9bf5277c36a23a239a82f` |
| `my_region` | 3,301,376 | `ef51d5e4a5206b9562edc5cc83f9e2bc26daa85438d94a33751161b55f07b5df` |
| `vendor_boot` | 100,663,296 | `457728c544e4e162b8fea3fa4d71c975279471c82ade62fdbec2b0c707107fc5` |
| `init_boot` | 8,388,608 | `eef84ca8aef1ced199b384cb5776b433893c0a0ee75030b38a706c9df39b45b5` |
| `vendor_dlkm` | 141,381,632 | `c6acc532c38be104942b2725aeaeb4acb27bf2cad5500fd2cd10ba16127ce313` |
| `dtbo` | 25,165,824 | `18d906153bc76c0fdbd6edd9899383e3fc898c17b8affcadc781024002108768` |

这只证明下载、payload 解析和选定镜像的主机侧一致性，不等于当前 A 槽逐块哈希，也不建立任何刷写授权。

### 指纹

| 观察 | `.501` 文件证据 | 与冻结源码的关系 |
|---|---|---|
| UFF 服务 | `vendor.oplus.hardware.biometrics.fingerprint@2.1-service_uff`，SHA-256 `4a66687057edd8d504dfd5145e50c1adc40f8adb640c1a21806edaeaea0d81c5`；RC 以 `system` 用户、`system input uhid` 组启动；VINTF 声明 AIDL fingerprint V3 | common 的 blob fixup 会给此服务注入 v3 `.oplus` shim；原厂 ELF 自身没有该自定义 shim 的 `NEEDED` 条目 |
| HAL 状态机线索 | ELF 字符串可见 `waitUiready`、`postUiready`、`onUiReady`、finger down/up 和 `ro.boot.vbmeta.device_state` | 同时支持“UI-ready 是当前直接门”和“boot-state 是次级候选”；不能仅凭字符串判断执行路径 |
| display 节点 | `init.oplus.display.rc` SHA-256 `d0283084117ef4f60ea17ca20ac9669587d47b0285c9cd603ec17dddf4adec56`；boot 时对 `notify_fppress` 执行 `chown system system` 与 `chmod 0666` | common proprietary list 包含此 RC，`init.oplus.rc` 明确导入它 |
| SELinux | 原厂文件提供节点初始化；冻结 `hardware/oplus` 把 `/kernel/oplus_display` 标成 `vendor_sysfs_graphics`，并允许 `hal_fingerprint_default` 读写该类 | 源码侧已有一条通用访问链，不应先验认定必须复制 zzkeier 的专用 type/rule |
| UFF 文件清单 | pagani 清单中的 20 个 `uff_jv`／`uff_spi` firmware 均存在；common phone 清单中的 UFF service、RC、AIDL V3 manifest 也均存在 | 不支持“缺 firmware/blob 导致 15001”的假设 |

主机侧最强结论是：现有源码已有 stock init 与通用 SELinux 基础，而已加载的 SensorProps shim 没有 FOD 事件桥。RandomLemon 的小范围源码移植因此比更换 firmware、vendor 或整棵 `hardware/oplus` 更符合现有证据；最终仍以实机日志为准。

### 2.4 GHz Wi-Fi

| `.501` 文件 | SHA-256 | 冻结 common 清单 |
|---|---|---|
| `odm/vendor/etc/wifi/WCNSS_qcom_cfg.ini` | `e4eb70484b0411a6728a73057db88dd4c36b6d13d0f1698ee5723fa6041206d9` | 已包含 |
| `odm/vendor/etc/wifi/WCNSS_qcom_cfg_roam.ini` | `406af0ea181b77d5104f85b10a3ece87e3a9a2ec981e2b5153a22d271600d960` | `9280681` 候选新增 |
| `odm/vendor/etc/wifi/WCNSS_qcom_cfg_cmcc.ini` | `ffc0698535573fc4bf0451de9eec5c24033693aadf1ed9d08f24fd69af7f4a49` | `9280681` 候选新增 |

原厂 `init.oplus.wifi.sh` 在普通路径选择基础 `WCNSS_qcom_cfg.ini`；只有 `persist.vendor.oplus.engineer.test=0/1/3` 才选 roam 文件。另一份原厂 `vendor.autochmod.sh` 可根据 `oplus.wifi.roaming.enabled` 选择 cmcc／基础文件，但冻结 Lineage、ABNOTF、zzkeier、Neveark 和 renhiyama 的所查源码／专有清单均没有这份 helper。`9280681` 增加的 firmware 搜索目录是 `/odm/etc/wifi/`，两个新增配置实际位于 `/odm/vendor/etc/wifi/`。

因此此次只能把它保留为“文件来源已确认、运行机制未闭合”的候选。设备测试应同时记录 2.4/5 GHz、连接协商、吞吐、重连、相关属性和 `/mnt/vendor/persist/wlan/WCNSS_qcom_cfg.ini` 的实际来源／哈希。

### 相机

- [`kinginu/oneplus13t-fix-camera@076bf52`](https://github.com/kinginu/oneplus13t-fix-camera/tree/076bf52e7d9b88b5620f83efe7276847cb20a18c) 是其 `CPH2723_16.0.3.501` 标签：`module/odm` 共 101 个文件，在本次 PKX110 `.501` ODM 中 101/101 存在且 101/101 SHA-256 相同。
- 冻结 [`OnePlus-13T-Development/android_device_oneplus_pagani@dfe9aa4`](https://github.com/OnePlus-13T-Development/android_device_oneplus_pagani/tree/dfe9aa41e8154fc89dc217efae564bac6c376216) 的 `proprietary-files.txt` 有 916 个非注释相机相关条目，本次提取分区中 916/916 路径存在。
- 这解释了该模块为什么能作为 ColorOS/OxygenOS 转换的 ODM 来源参考，也说明当前 pagani 清单没有明显的整批相机文件空洞；它不验证 OPlus camera framework、HDR、变焦、夜景或长时间录像。

### 运营商与信号格

- `.501` 的 `CarrierConfigResCommon_Sys.apk` SHA-256 为 `336bdf48e7c91e7fb594ecdbe91f611d9c24e44b602964f5588c9d6039021559`。解出的中国运营商 LTE RSRP 表与冻结 Lineage common 对应块一致；部分 `.501` 差异是 NR availability 等其他键，不是中国 NR 信号格门限。
- `.501` 的 `OplusCarrierConfig.apk` SHA-256 为 `a6e7694ab3004d7fe67f5ca834d81eb5c222494e89cfe54fb8038edfa2536dbf`。其中确有 `5g_nr_ssrsrp_thresholds_int_array`／`5g_nr_sssinr_thresholds_int_array`，但命中的是 SoftBank/KDDI 等日本运营商块；CMCC、CU、CT、CBN 块没有同类表。
- `my_product/build.prop` 有 `ro.oplus.radio.lte_rsrp_thresholds=-126,-121,-114,-105,-44`。单看属性名称不能证明 NR 行为；后续从同版本 `oplus-telephony-common-ext.jar` 确认 `getNrLevel()` 直接调用 `getLteLevel()`，才闭合了 NR 复用这组边界的证据链。
- `CarrierConfigOverlay.platform.8750B.apk` 中的 RSRP/RSRQ/SINR 表名称属于 `oplus_carrier_ims_rtp_redun_*`，用于 IMS RTP 冗余／媒体质量判断，不是状态栏信号格。

本页最初的 CarrierConfig APK 审计没有产生可直接移植的“中国 NR 门限表”；该判断现已被更深一层的 OPlus framework 审计补充。AOSP 四门限数组采用 `[-126, -121, -114, -105]`，`-44` 仅作为有效上限。修复和刷入前后 60 轮实机对比见[中国 NR 信号等级修复报告](china-nr-signal-fix-2026-09-01.md)。仍不应改 modem 或把日本运营商、IMS 媒体门限挪作中国状态栏门限。

## 现有问题的处理顺序

| 优先级 | 问题 | 当前判断 | 下一步 |
|---|---|---|---|
| P0 | 指纹录入 `15001` | UI-ready／OFP 事件桥缺口最符合日志 | 做上述最小候选构建与实机回归 |
| P1 | 2.4 GHz Wi-Fi | 原厂文件存在，但候选加载机制与当前是否复现都未知 | 分别记录 2.4/5 GHz 连接、吞吐、重连与实际配置加载 |
| DONE | NR 信号格偏低 | `.501` OPlus framework 的 NR 等级算法已确认，MCC 460 修复通过完整构建和实机验证 | 保持当前映射；后续只补外国 SIM、漫游和专用覆盖测试 |
| P1 | GApps / Google Clock | priv-app allowlist 与安装流程问题 | 在干净构建中修权限并复测，不归因于 device tree |
| P2 | 相机完整功能矩阵 | 四设备与基础拍摄已通过；ColorOS 功能不等于 AOSP 基础链 | 保留当前栈，只补实拍、HDR、变焦与长时录像测试 |
| P2 | HDR display_config 线索 | 跨设备社区线索，尚无 pagani 复现 | 仅在实际 HDR 显示异常时定向核对原厂配置 |

## 本页不作出的结论

- 不宣称 RandomLemon、zzkeier、ABNOTF、renhiyama 或 Neveark 的指纹方案已在当前 PKX110 上可用。
- 不把 `.501` 公开 OTA 等同于用户 A 槽的实时、个性化分区副本。
- 不以原厂文件存在证明 AOSP 一定需要复制该文件。
- 不建议替换完整 device tree、`hardware/oplus`、vendor、ODM、camera 或 firmware。
- 不触碰 ABL、XBL、TZ、HYP、AOP、modem、DSP 等 ARB／启动链高风险分区。
