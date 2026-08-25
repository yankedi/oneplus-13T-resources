# 本地 LineageOS 构建审计（2026-08-25）

## 范围与结论

本次只读检查了以下本地源码和构建输出：

- `<LINEAGE_ROOT>`：LineageOS 23.2 Android 16 源码树，包含 `.repo/manifest.xml`、`build/envsetup.sh`、`device/oneplus/pagani` 和 `out/target/product/pagani`。
- `<LINEAGE_OUT>`：该源码树的 active 输出目录。
- 资料库此前记录的 2026-08-14 完整 OTA 及 2026-08-16 的 UDFPS 增量构建记录。

本次没有 `repo sync`、清理、重新编译、刷机、重启或手机操作。源码树和 active 输出均保留原状。

结论先行：

- `HOST_VERIFIED`：完整 OTA `lineage-23.2-20260814-UNOFFICIAL-pagani.zip` 与已有记录完全一致，大小和 SHA-256 均匹配。
- `HOST_VERIFIED`：该 OTA 的 payload 签名、target-files AVB chain、VINTF 和动态分区尺寸检查均通过；payload 的十六进制 SHA-256 为 `bc1168712e2fc5f26c5ea03dbe48b5c71f935b0471e3adfb8eaf2322898876de`。
- `HOST_VERIFIED`：active `out` 已被 2026-08-16 的 dirty 增量构建部分改写。当前 `system_ext.img` 与旧 `vbmeta.img` 的 hashtree 不匹配，不能把 active `out` 当作完整、可刷写的镜像集合。
- `SOURCE_VERIFIED`：当前源码没有 `OplusFodShim` 文件，但 dirty 的 SystemUI `UdfpsController` 已加入直接写入 `notify_fppress` 的事件桥；同时 pagani/common、SystemUI 和 QTI display 项目均有未提交修改。
- `NOT_TESTED`：本次没有真机验证这些 dirty 增量内容。资料库已有的指纹录入失败结论继续有效，不能因为主机输出包含策略或 APK 就升级为功能修复。

## 与既有记录的关系

### 已独立重新确认

以下六个构建基线 commit 由本地 `.repo/local_manifests/pagani.xml`、各 Git checkout 的 `HEAD` 和生成的 target-files manifest 交叉确认，均与资料库记录一致：

| 项目 | 公开仓库 | 本地 HEAD | 结果 |
|---|---|---|---|
| `device/oneplus/pagani` | [`OnePlus-13T-Development/android_device_oneplus_pagani`](https://github.com/OnePlus-13T-Development/android_device_oneplus_pagani) | `dfe9aa41e8154fc89dc217efae564bac6c376216` | `SOURCE_VERIFIED`，重新确认 |
| `device/oneplus/sm8750-common` | [`LineageOS/android_device_oneplus_sm8750-common`](https://github.com/LineageOS/android_device_oneplus_sm8750-common) | `44ad18fa12b51983a36fb7ec67c54a6b4c032859` | `SOURCE_VERIFIED`，重新确认 |
| `hardware/oplus` | [`LineageOS/android_hardware_oplus`](https://github.com/LineageOS/android_hardware_oplus) | `ec3b8211676f55ed09905c4c336356acedc040d3` | `SOURCE_VERIFIED`，重新确认 |
| `kernel/oneplus/sm8750` | [`LineageOS/android_kernel_oneplus_sm8750`](https://github.com/LineageOS/android_kernel_oneplus_sm8750) | `999b95d4792dd41fba72a7914abe897cc3ee2ecd` | `SOURCE_VERIFIED`，重新确认 |
| `kernel/oneplus/sm8750-devicetrees` | [`LineageOS/android_kernel_oneplus_sm8750-devicetrees`](https://github.com/LineageOS/android_kernel_oneplus_sm8750-devicetrees) | `ebb25e3526ad84cc5a1090a5a9242f33ff087bf2` | `SOURCE_VERIFIED`，重新确认 |
| `kernel/oneplus/sm8750-modules` | [`LineageOS/android_kernel_oneplus_sm8750-modules`](https://github.com/LineageOS/android_kernel_oneplus_sm8750-modules) | `eab0a34a8b657ee22839c9b3878c3b6e687675f9` | `SOURCE_VERIFIED`，重新确认 |

所有 repo 项目均处于 detached `HEAD`；manifest 默认分支是 `lineage-23.2`，AOSP remote 固定为 `android-16.0.0_r4`。源码根本身是 repo client，没有普通顶层 Git repository。生成的 build manifest 还确认 Lineage superproject revision 为 `d38d2e178902482ea4b9da171b63ce640ce6e473`，`build/make` revision 为 `5a841be38fec92de9a8a408cf5812ca748ccc02e`。

### 本地 dirty 状态

`SOURCE_VERIFIED`：repo status 报告以下项目有未提交修改。摘要只描述实现，不保存完整 diff。

| 项目 | HEAD | 状态 | `git diff --stat` | 功能相关摘要 |
|---|---|---|---|---|
| `device/oneplus/pagani` | `dfe9aa41e8154fc89dc217efae564bac6c376216` | dirty；3 个 tracked 文件，4 个未跟踪 sepolicy 文件 | `BoardConfig.mk` 4+0；`device.mk` 1+0；`proprietary-files.txt` 1+6 | 接入 pagani sepolicy；打开 `qtidisplay.oplus_udfps`；调整 camera AIDL prebuilt suffix/重复项 |
| `device/oneplus/sm8750-common` | `44ad18fa12b51983a36fb7ec67c54a6b4c032859` | dirty | `init/ueventd.oplus.rc` 1+0 | 为 `notify_fppress` 增加 `system:system`、`0664` 节点规则 |
| `frameworks/base` | `aaa4284f7e061771afb58c789924397111487e62` | dirty | `UdfpsController.java` 23+0 | finger down/up 和 overlay hide 路径写入 `/sys/kernel/oplus_display/notify_fppress` |
| `hardware/qcom-caf/sm8750/display/core` | `20cf597e21bdd31af4e3a55660e991e22f69bf8b` | dirty | 6 个文件，74+0 | 增加 `CONNECTOR_SET_FINGERPRINT_MASK`，映射 `fingerprint_mask`/`hbm_enable` 并提交 connector atomic property |
| `hardware/qcom-caf/sm8750/display/hal` | `4b74f47925c54e95c805275832a65830af0431b6` | dirty | `Android.bp` 9+0 | 为 `oplus_udfps` 打开 `CONNECTOR_PROP_UDFPS` 和 `OPLUS_FINGERPRINT_MASK` 编译宏 |

`SOURCE_VERIFIED`：以下项目在本次检查时 clean：`hardware/oplus`、三个 kernel 项目。`vendor/oneplus/pagani` 和 `vendor/oneplus/sm8750-common` 是由 proprietary extraction 生成的目录，本地未发现独立 Git metadata，因此没有 vendor commit 可记录；构建日志确认它们被实际纳入增量构建。未跟踪文件和 proprietary 内容均未复制到本仓库。

## 构建环境与记录

### 主机环境

以下是本次审计主机观测值；没有记录用户名、主机名或真实绝对路径。

| 项目 | 观测值 |
|---|---|
| OS | NixOS 26.05 (Yarara)，build `26.05.20260823.a3b9886` |
| Host kernel/arch | Linux 7.1.10，x86_64 |
| CPU | AMD Ryzen 9 7945HX with Radeon Graphics，16 cores / 32 threads |
| Memory/swap | 30 GiB / 46 GiB |
| Host Git | 2.54.0 |
| Host Java | OpenJDK 25.0.4 |
| Host Python | 3.13.15 |
| Nix | 2.34.8 |
| repo launcher | 2.65；源码内置 launcher 可用，宿主默认 PATH 中没有独立 `repo` 命令 |

### FHS 构建环境

`HOST_VERIFIED`：现存成功构建和增量构建均使用 `<BUILD_ENV>` FHS wrapper。当前 wrapper 观测到 ccache 4.13.6、Git 2.55.0、OpenJDK 25.0.4、Python 3.14.7。ccache 已启用，配置上限为 20.0 GB，当前统计为 9.5/20.0 GB（47.59%）、87,434 次可缓存调用、命中率 30.37%；磁盘占用约 8.9G。

成功完整构建记录的 target 是 `lineage_pagani-bp4a-userdebug`，产品为 `lineage_pagani`，variant 为 `userdebug`，架构为 `arm64` / `armv8-2a-dotprod`。可靠记录中的完整构建命令是 `mka bacon -j8`，日志记录耗时 `20:54`。本次没有重跑该命令。

2026-08-16 dirty 增量构建的实际命令记录为：

```sh
source build/envsetup.sh
lunch lineage_pagani-bp4a-userdebug
m SystemUI vendorimage -j8
```

该轮日志记录 `LINEAGE_VERSION=23.2-20260816-UNOFFICIAL-pagani` 和耗时 `11:25`。随后 host-only image 轮次同样为 Android 16、`lineage_pagani`、`userdebug`，日志记录耗时 `04:11`，构建了 `system_ext.img`、`vendor.img` 和 `recovery.img`。这些命令没有重新生成完整 OTA。早期 `lineage_pagani-userdebug` lunch 名称被 Android 16 parser 拒绝，已排除为当前有效 target。

### Android / Lineage 属性

`HOST_VERIFIED`：完整 target-files 的属性为 Android 16、SDK 36、安全补丁 `2026-08-01`、LineageOS `23.2-20260814-UNOFFICIAL-pagani`。构建内部 fingerprint 已将 builder 字段脱敏为：

```text
OnePlus/lineage_pagani/pagani:16/BP4A.251205.006/eng.<REDACTED>:userdebug/test-keys
```

系统产品 fingerprint 为 `OnePlus/PKX110/OP60F5L1:16/AP3A.240617.008/V.1d6b086_3e842d_3e842a:user/release-keys`。target-files kernel version 是 `6.6.142-4k-g999b95d4792d`。

## 构建产物

### 完整 OTA

`HOST_VERIFIED`：下面的完整 OTA 是本次实际读取和重新哈希的文件。它与既有记录相同，不是 2026-08-16 dirty 增量构建的新包。

| 文件 | 大小（bytes） | SHA-256 | 构建/文件时间 |
|---|---:|---|---|
| `lineage-23.2-20260814-UNOFFICIAL-pagani.zip` | 2503659880 | `bcdec8cbcf5b42f42ff2e998ecb8f49bab6316fdc5d36d426725f0ee12bdde8e` | 2026-08-14 17:38:17 +0800 |
| embedded `payload.bin` | 2503651364 | `bc1168712e2fc5f26c5ea03dbe48b5c71f935b0471e3adfb8eaf2322898876de` | OTA 内 |

`HOST_VERIFIED`：`lineage_pagani-ota.zip` 与正式命名 ZIP 是同一 inode 的硬链接，大小和 SHA-256 完全相同，不是第二次构建。OTA 为 A/B payload OTA。未提交 ZIP、payload 或任何镜像。

### Coherent target-files image set

`HOST_VERIFIED`：以下是 target-files `IMAGES/` 中与上述完整 OTA 同一时间点的镜像。它们通过 `avbtool verify_image --follow_chain_partitions`；时间列按本地文件时间记录到日期，精确时间只用于本地追溯。

| 文件 | 大小（bytes） | SHA-256 | 日期 |
|---|---:|---|---|
| `vbmeta.img` | 12288 | `fc80167cbcd058bd8d959eb1e046abf499c1216be1f468999ff6ac04d0867c1e` | 2026-08-14 |
| `vbmeta_system.img` | 4096 | `d70cd3f8a19620447ee377b56618c4e6c3974bf8e284cda8ceb9d317eb512b7d` | 2026-08-14 |
| `vbmeta_vendor.img` | 4096 | `3b173dbfae537f0bef05b3cc523c4c30d22516ea5504d0275513c49f534b9811` | 2026-08-14 |
| `boot.img` | 100663296 | `c9a6b0e2a2fbe9e0b8f64d061220896910fb92018d4d821142e7fa6d4066c7b6` | 2026-08-14 |
| `init_boot.img` | 8388608 | `7a6ffaa97c0c1c1a172dd7669aac1617c5befd3156351efc99f47374fe972ef8` | 2026-08-14 |
| `vendor_boot.img` | 100663296 | `7e76b3b6e557fdca8d9cd3a8520a9cf3e239cc35b1b2f25e73f193a0d392998d` | 2026-08-14 |
| `dtbo.img` | 25165824 | `099c7b3b81ff934a3ef739cb38124f1f4b6033ed82fb2063329fa7b69f2f8f49` | 2026-08-14 |
| `recovery.img` | 104857600 | `e136fd6549526a92e2ac1c805c80d7762418448824f4e2d9ec11209dbfa9bef7` | 2026-08-14 |
| `system.img` | 987230620 | `a3e08c2bdd2b771c438172ecfb164879aaa2e79fad894bfb1112303c95beddc5` | 2026-08-14 |
| `system_ext.img` | 472719628 | `5a35a633fa048063b81beb5f9b775322f4647cb6040be08e68398b3214a21b6f` | 2026-08-14 |
| `vendor.img` | 364621824 | `beaef9cb273e4b1f7845750e2eb15982dc471c2feed72ef24a47bedb5c96dd9b` | 2026-08-14 |
| `odm.img` | 1702928384 | `01133d2fe5bd0d5e3104d7e60541b33361a3400c02ff6879c5d64666fe65574d` | 2026-08-14 |
| `product.img` | 544952672 | `f1282551dc4f9252069d7289aa5f53fb3c2b35c3cd072b4e115841b188bc2a53` | 2026-08-14 |
| `system_dlkm.img` | 7512064 | `2c55b85d95486bd7349561f96c09412bd78a5656e0a88beae27312df1c5654b8` | 2026-08-14 |
| `vendor_dlkm.img` | 41463808 | `f00d00739e452bbc767e9e4608a00f1ec808a1b939b53a1daa8507fde3168e33` | 2026-08-14 |

### Active output after dirty increments

`HOST_VERIFIED`：下面的文件实际存在于 active `out/target/product/pagani`，但不属于一个新打包 OTA。`system_ext.img`、`vendor.img` 和 `recovery.img` 的文件时间来自 2026-08-16 增量 image 构建；旧 `vbmeta.img` 仍来自 2026-08-14。

| 文件 | 大小（bytes） | SHA-256 | 文件时间 |
|---|---:|---|---|
| `system_ext.img` | 472715532 | `84d82050af7cbeb4af4778e6d2d8e5e4c699e3ef0702640efc8bacea470a7c96` | 2026-08-17 07:11:26 +0800 |
| `vendor.img` | 364711936 | `4b6b7cee9890b055633547e8b16cde0666ab780c6d07b8f695e910bb71e9290e` | 2026-08-17 07:12:22 +0800 |
| `recovery.img` | 104857600 | `1259462271444f1746eab4601778474dc33fc1a054b7fd6e37954c771325fdac` | 2026-08-17 07:12:18 +0800 |
| `system_ext/priv-app/SystemUI/SystemUI.apk` | 41039834 | `411bad55abb94faa0bc884b0c95ded706f4d96883eb25c2b30196a61da219635` | 2026-08-17 00:05:53 +0800 |
| `vendor/etc/selinux/vendor_sepolicy.cil` | 2202542 | `b08851eef1e494d765c630d9313597fa13812f238b1e908c7ec43ebe29b90d1c` | 2026-08-16 23:58:57 +0800 |

## 主机侧验证

| 检查 | 结果 | 证据与边界 |
|---|---|---|
| OTA whole-package / A/B payload signature | `HOST_VERIFIED` PASS | FHS 内 `check_ota_package_signature`；OpenSSL 3 的 `rsautl` deprecation warning 不影响结果 |
| OTA ZIP entry CRC | `HOST_VERIFIED` PASS | Python `zipfile.testzip()` 返回无坏 entry |
| Info-ZIP `unzip -t` | `HOST_VERIFIED` WARN | Android SignApk ZIP 的 6 个短 extra-field 被 Info-ZIP 报格式警告；不以该工具退出码取代 payload/signature 结果 |
| coherent target-files AVB chain | `HOST_VERIFIED` PASS | root vbmeta、boot、dtbo、recovery、vendor_boot、init_boot、vbmeta_system/vendor 及动态分区 hashtree 均通过 |
| active output AVB chain | `HOST_VERIFIED` FAIL | `system_ext.img` hashtree 不匹配旧 `vbmeta.img` descriptor；这是混合输出证据，不是对完整 OTA 的否定 |
| VINTF | `HOST_VERIFIED` PASS | 构建生成的 `check_vintf_compatible.log` 为 `COMPATIBLE`；带 generated APEX map 的独立 host invocation 也返回 0 |
| dynamic partition fit | `HOST_VERIFIED` PASS | 7 个动态分区内容 `6334275584 <= 13329498112`；含 1 MiB metadata 仍满足 super size |

target-files 配置为 dynamic partitions `odm product system system_dlkm system_ext vendor vendor_dlkm`、Virtual A/B、`virtual_ab_cow_version=3`，`vintf_enforce=true`，AVB 算法为 SHA256_RSA4096，root vbmeta flags 为 `3`。这些是主机侧包一致性结果，不等于设备启动、userdata 安全或真机功能通过。

## 功能线索审计

### 屏下指纹与 `notify_fppress`

- `SOURCE_VERIFIED`：pagani overlay 已提供 UDFPS enrollment 进度资源和 `org.lineageos.sensor.udfps` long-press sensor；common init 导入 stock-derived `init.oplus.display.rc`。
- `SOURCE_VERIFIED`：`hardware/oplus/fingerprint/shims/SensorPropsShim.cpp` 读取 `persist.vendor.fingerprint.*` 和 DRM 模式，生成 sensor location/radius，并把 UDFPS touch 交给 HAL；当前 `hardware/oplus/fingerprint/` 没有 `OplusFodShim.cpp`。
- `SOURCE_VERIFIED`：dirty `UdfpsController.java` 在首次 finger down 写 `1`，finger up 和 hide overlay 写 `0`，并以 `mFppressActive` 避免重复写；这是 framework 侧直接 writer，不是 Oplus hardware shim。
- `SOURCE_VERIFIED`：dirty QTI display core/hal 增加 `fingerprint_mask`/`hbm_enable` connector property，pagani `device.mk` 打开 `qtidisplay.oplus_udfps`。
- `HOST_VERIFIED`：active output 的 `vendor_sepolicy.cil` 含 `sysfs_notify_fppress` type、genfs label 和 `platform_app_202504` 的 `{ write getattr open }`，hash 为 `b08851...`；active SystemUI APK hash 为 `411bad...`。这些只证明 dirty 增量进入了 host output。
- `HOST_VERIFIED`：output 中的 stock-derived display init rc 对该节点执行 `chown system system` 与 `chmod 0666`，hash 为 `d0283084...`；common 源码导入该 rc。
- `NOT_TESTED`：本次没有加载 dirty image 到设备，没有检查运行时 SELinux context/AVC、writer domain、`session.onUiReady()`、HBM、AOD、取消和休眠回归。既有 `DEVICE_VERIFIED` 的 enrollment failure 仍不能由这次主机检查改写。

### 相机

- `SOURCE_VERIFIED`：pagani proprietary list 包含 ODM camera model/config、`cammidasservice`、RFI/sendextcamcmd service implementation，以及 vendor QTI camera provider；common phone list 包含 IMS 与相关系统扩展依赖。
- `HOST_VERIFIED`：输出包含 `vendor.qti.camera.provider-service_64`、`ICameraProvider/vendor_qti/0` VINTF、offline camera/AON/RFI interface 声明和多个 OPlus camera VINTF 条目。
- `NOT_TESTED`：本次没有运行相机、枚举、录像、HDR、变焦或长时间稳定性测试。源码和产物存在不等于相机功能正常；既有实机基础 camera 记录保持独立。

### 2.4 GHz Wi-Fi

- `SOURCE_VERIFIED`：当前 common HEAD 是 `44ad18fa...`，本地 dirty 变化只增加 `notify_fppress`，没有包含 `zzkeier/android_device_oneplus_sm8750-common@92806812c82f10d42ea663c1b6348c2a97294d7b` 的新增 proprietary 配置。
- `HOST_VERIFIED`：当前 output 的 ODM 文件清单只出现 `odm/vendor/etc/wifi/WCNSS_qcom_cfg.ini`；没有 `WCNSS_qcom_cfg_roam.ini` 或 `WCNSS_qcom_cfg_cmcc.ini`。
- `NOT_TESTED`：本次没有分频段连接、吞吐、重连或实际 persist 配置来源测试；既有 `9280681` 候选的排重和未验证状态不变。

### 移动网络

- `SOURCE_VERIFIED`：common overlay 将 5G NR SA/NSA 和 WFC over IMS 设为可用；network manifest 声明 QTI IMS AIDL v16，以及 OPlus `OplusImsRadio0/1` 和 `OplusRadio0/1`。
- `HOST_VERIFIED`：输出包含 IMS framework/vendor libraries、IMS daemons、QTI camera/provider 及 ODM OPlus radio/IMS VINTF instances。
- `NOT_TESTED`：本次没有 SIM、信号格、NR SA/NSA、IMS、VoLTE/VoNR 或 CarrierConfig 运行时测试。`.501` 中国 NR 门限“没有直接答案”的既有结论不变。

## 设备树实际差异与排重

`SOURCE_VERIFIED`：本地构建树与冻结成功构建基线的实际变化集中在：

- pagani：sepolicy 接入、`qtidisplay.oplus_udfps`、camera AIDL prebuilt 冲突处理；
- common：`notify_fppress` ueventd 权限；
- frameworks/base：SystemUI 直接 writer；
- QTI display core/hal：fingerprint mask connector property 和编译宏。

`SOURCE_VERIFIED`：本地没有应用 ABNOTF、zzkeier、renhiyama、Neveark 或 RandomLemon 的整棵设备树。`9280681` 的 Wi-Fi 改动、RandomLemon 的 `OplusFodShim`、候选 SystemUI patch 及其他设备树差异已经在既有 [设备树比较](device-tree-comparison.md) 和 [候选参考目录](pending-references.md) 中登记；本次不重复创建相同来源或相同结论。

`NOT_TESTED`：任何候选实现仍未因源码存在、dirty 增量成功或主机哈希而变成设备支持结论。

## 隐私与未提交材料

以下内容明确未提交：

- 源码 checkout、`.repo`、vendor tree、完整 build logs、target-files、OTA ZIP、payload、boot/system/vendor/odm 镜像和任何 proprietary blob；
- builder username、hostname、真实家庭目录、设备 serial、IMEI/IMSI/ICCID、手机号、Wi-Fi 标识和认证材料；
- signing key、keystore、keybox、token、Cookie、提取码及完整运行时日志；
- 仅保留必要的文件名、大小、哈希、公开仓库 URL、源码摘要和脱敏错误/警告。

## 后续仍需验证

- `NOT_TESTED`：只使用同一构建代际重新打包后的完整镜像集合，不能使用当前 active mixed output；重新打包前应先处理或明确 dirty 状态。
- `NOT_TESTED`：在可回滚条件下确认 `notify_fppress` 节点存在性、实际 context、SystemUI writer domain 和 AVC，再验证 UI-ready、HBM、AOD、取消、休眠和手势路径。
- `NOT_TESTED`：2.4/5 GHz 分频段 Wi-Fi 及实际配置来源；不能仅凭 `9280681` 文件差异判定修复。
- `NOT_TESTED`：相机实拍、HDR、变焦、录像和长时间稳定性；不能从 provider/VINTF 清单推导功能正常。
- `NOT_TESTED`：CarrierConfig 运行时选择和中国 NR 信号格映射；不能把日本运营商数组或 IMS RTP 参数当作中国 NR 门限。
