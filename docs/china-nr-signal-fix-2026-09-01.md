# OnePlus 13T 中国 NR 信号等级修复与实机验证

更新时间：2026-09-01

## 结论

OnePlus 13T 国行 `.501` 的 OPlus telephony framework 明确使用
`-126,-121,-114,-105,-44` 计算 LTE 信号等级，并让 NR SSRSRP 复用同一算法。
LineageOS 原先使用 AOSP 默认 NR 门限 `[-110, -90, -80, -65]`，导致约
`-90 dBm` 的正常 NR SA 信号只报告 level 2。

修复把 OPlus 算法的前四个等级边界按 AOSP CarrierConfig 格式写入 MCC 460：

```text
[-126, -121, -114, -105]
```

完整构建和 slot B 定向部署通过。刷入后两个订阅的有效 CarrierConfig 都采用新门限，
连续 60 轮采样均报告 level 4。该结论标记为 `SOURCE_VERIFIED + HOST_VERIFIED + DEVICE_VERIFIED`。

## 门限来源

来源是公开 `.501` 完整 OTA 中的：

```text
PKX110_16.0.3.501(CN01)
system_ext/framework/oplus-telephony-common-ext.jar
SHA-256: bb240ae2059deac35d69d946998b23c523b6dd9c8e0b5e135bb3141b392a5529
```

反编译后的 `com.oplus.internal.telephony.OplusSignalStrengthStandard` 显示：

- `ro.oplus.radio.lte_rsrp_thresholds` 默认值为 `-126,-121,-114,-105,-44`；
- `getNrLevel(ssRsrp)` 直接调用 `getLteLevel(ssRsrp)`；
- 四级显示边界依次为 `-126`、`-121`、`-114`、`-105 dBm`；
- `-44 dBm` 是有效 RSRP 上限，不是第五个等级边界。

对应的 OPlus 四格映射为：

| SSRSRP | level |
|---|---:|
| `< -126 dBm` | 0 |
| `-126 .. -122 dBm` | 1 |
| `-121 .. -115 dBm` | 2 |
| `-114 .. -106 dBm` | 3 |
| `-105 .. -44 dBm` | 4 |

AOSP 的 `5g_nr_ssrsrp_thresholds_int_array` 只接受四个有序边界，合法范围为
`[-140, -44]`，因此只移植前四个值。对 `-140 .. -44 dBm` 的全部整数输入执行了
OPlus 算法与 AOSP 四门限算法的等价性检查，结果通过。

这修正了早期只检查 CarrierConfig APK 后得出的不完整判断：`.501` 的中国运营商
CarrierConfig 块确实没有直接列出 NR 表，但 OPlus telephony framework 已明确规定
NR 复用上述 LTE 等级算法，因此该组中国 ROM 实际行为可以被确定。

## 源码实现

修复提交：[`yankedi/android_device_oneplus-13T_common@42e94f0`](https://github.com/yankedi/android_device_oneplus-13T_common/commit/42e94f0e61f63c9622a6bfc90878098da682f7f1)

路径：

```text
device/oneplus/sm8750-common/overlay/CarrierConfigResCommon/res/xml/vendor.xml
```

配置：

```xml
<carrier_config mcc="460">
    <int-array name="5g_nr_ssrsrp_thresholds_int_array" num="4">
        <item value="-126" />
        <item value="-121" />
        <item value="-114" />
        <item value="-105" />
    </int-array>
</carrier_config>
```

该规则只匹配中国 MCC 460。其他国家的 MCC/MNC 配置和 AOSP 默认值没有修改；
更具体的运营商配置仍可覆盖该通用块。CarrierConfig 主要按订阅归属选择，所以中国
SIM 在境外漫游时通常仍使用 MCC 460 配置，外国 SIM 不会使用这组门限。

源码仓库统一使用 `lineage-23.2` 作为构建和提交分支。此前的
`pagani-fingerprint-fix` 只是合并前的临时分支；其提交已经进入主分支，构建不依赖该分支名。

## 构建与产物验证

| 项目 | 结果 |
|---|---|
| `CarrierConfigResCommon` 单目标构建 | PASS |
| 完整 `mka bacon -j8` | PASS |
| whole-package 与 A/B payload 签名 | PASS |
| 最终 payload 中 CarrierConfig APK 与构建产物哈希 | MATCH |
| VINTF、SELinux、动态分区容量和 AVB 链 | PASS |

完整 OTA：

| 字段 | 值 |
|---|---|
| 文件 | `lineage_pagani-ota.zip` |
| 大小 | `2,503,306,005` bytes |
| SHA-256 | `9b612a1334e3c9fc419755154572b63afe27d901f4a4a53117cb5de0c2be0cff` |
| CarrierConfig APK SHA-256 | `c94004d76050c7f451ba8f70e4dfe5dbacde7011ae011a46518053302fa43bbc` |

## 实机部署与前后对比

设备当时运行 slot B。为保护 slot A 的 ColorOS 救援系统，仅写入：

- `product_b`，SHA-256 `47d2e5ce6256cef5e997b2250ccc6d8aa3094f68e2b34b946139c5b6af14e81b`；
- `vbmeta_system_b`，SHA-256 `7202bf24f2a96cb9c1f634f2ae0c69e511194cf479aa3cf8734b2c6cde6da661`。

slot A 和其他分区均未写入。设备从 slot B 正常启动，`sys.boot_completed=1`。

| 项目 | 刷入前 60 轮 | 刷入后 60 轮 |
|---|---:|---:|
| 有效 NR SSRSRP 门限 | `[-110, -90, -80, -65]` | `[-126, -121, -114, -105]` |
| Phone 0 framework level | 60/60 为 2 | 60/60 为 4 |
| Phone 1 framework level | 60/60 为 2 | 60/60 为 4 |
| 驻网 | NR SA，60/60 在网 | NR SA，60/60 在网 |
| 当前数据连接 | Phone 0，60/60 已连接 | Phone 0，60/60 已连接 |

刷入前 Phone 0/1 平滑 SSRSRP 均为约 `-90 dBm`；刷入后分别为约 `-87 dBm`
和 `-85 dBm`。射频数值会随环境变化，因此这里只验证 CarrierConfig 和 framework
等级映射生效，不把信号格增加解释为天线、基带或实际射频性能提升。

## 边界

- 这是状态栏所消费的 framework 信号等级修复，不增强接收能力、吞吐或通话质量。
- 中国 MCC 460 已实机验证；外国 SIM、境外漫游和运营商专用覆盖尚未专项测试。
- 原始设备日志含订阅和网络信息，不进入公开仓库；本文只保留脱敏统计、版本、哈希和结论。
