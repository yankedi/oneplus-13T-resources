# AOSP / LineageOS 差异审计

最后核对：2026-08-24

> **后续状态（2026-08-30）**：本页的 UDFPS 缺口判断属于修复前快照。真实触摸驱动的 `notify_fppress` 路径、精确 SELinux 规则和 SystemUI cleanup-only writer 已完成构建与实机验证；当前结论见[指纹修复与实机验证](fingerprint-fix-2026-08-30.md)。其余子系统审计仍保持原时间截面。

本页回答一个窄问题：当前已成功启动的 LineageOS 23.2 源码，与 OnePlus 13T 社区参考实现相比，哪些差异值得继续验证。它不是把另一个设备树整体移植过来的清单。

## 两个不能混用的时间截面

### 已验证构建清单

2026-08-14 生成、随后在 PKX110 上完成启动和硬件冒烟测试的构建使用以下固定提交：

| 仓库 | 提交 |
|---|---|
| `OnePlus-13T-Development/android_device_oneplus_pagani` | `dfe9aa41e8154fc89dc217efae564bac6c376216` |
| `LineageOS/android_device_oneplus_sm8750-common` | `44ad18fa12b51983a36fb7ec67c54a6b4c032859` |
| `LineageOS/android_hardware_oplus` | `ec3b8211676f55ed09905c4c336356acedc040d3` |
| `LineageOS/android_kernel_oneplus_sm8750` | `999b95d4792dd41fba72a7914abe897cc3ee2ecd` |
| `LineageOS/android_kernel_oneplus_sm8750-devicetrees` | `ebb25e3526ad84cc5a1090a5a9242f33ff087bf2` |
| `LineageOS/android_kernel_oneplus_sm8750-modules` | `eab0a34a8b657ee22839c9b3878c3b6e687675f9` |

这些提交描述的是“成功构建并经过设备验证的基线”，不应随上游 HEAD 自动改写。

### 历史参考快照

2026-08-16 的逐项差异审计冻结在以下 `16.0` 提交：

| 仓库 | 提交 |
|---|---|
| `Oneplus-13T-AOSP/android_device_oneplus_pagani` | `438f9c9e` |
| `Oneplus-13T-AOSP/proprietary_vendor_oneplus_pagani` | `b82c534b` |
| `Oneplus-13T-AOSP/android_device_oneplus_sm8750-common` | `faca90d7` |
| `Oneplus-13T-AOSP/android_hardware_oplus` | `7cd64aa6` |
| `Oneplus-13T-AOSP/android_vendor_extraPatches` | `5733d306` |
| `Oneplus-13T-AOSP/local_manifests` | `7f47256f` |

到 2026-08-23，多个参考仓库的默认分支已经转向 `seventeen`。因此，旧审计结论与当前 HEAD 必须分别记录；当前 HEAD 不能反向替代当时分析所依据的提交。

## 分类结果

完整的 30 项矩阵见 [`data/feature-matrix.tsv`](../data/feature-matrix.tsv)。汇总如下：

| 分类 | 数量 | 含义 |
|---|---:|---|
| `IDENTICAL` | 11 | 已存在同等实现，无需移植 |
| `MISSING` | 4 | 当前树缺少对应实现，但仍需先证明运行时需求 |
| `PARTIALLY_PORTED` | 6 | 两边存在共同基础，只能选择性审计 |
| `DIFFERENT_IMPLEMENTATION` | 5 | 架构或代际不同，不可直接复制 |
| `ALREADY_BETTER` | 1 | 当前共享实现更合适，保留现状 |
| `MANUAL_PORT_REQUIRED` | 1 | 补丁必须逐项重放和复核 |
| `NEEDS_DEVICE_TEST` | 1 | 暂无源码缺口指向，需要设备证据 |
| `DO_NOT_PORT` | 1 | 风险高于当前收益 |

## 建议的验证顺序

1. **UDFPS UI-ready / `notify_fppress` 链路（P0）**：现象是 HAL 等待约 502 ms 后返回错误 `15001`。`.501` 与冻结源码交叉结果表明 stock display rc 和 fingerprint HAL 的通用节点访问链已经存在，第一轮应保留当前 SensorProps，只定向移植 RandomLemon 的 `OplusFodShim`／构建接线；先采集节点上下文和 AVC，再决定是否需要新增规则。
2. **指纹手势抢占与休眠遮罩清理（P1）**：与主故障拆开验证，避免将生命周期修复和错误屏蔽混在一起。
3. **CarrierSettings / NR 信号格映射（P1）**：设备已能 NR SA 通话和数据；`.501` 的中国运营商块没有可直接移植的 NR 门限，不能把日本运营商或 IMS RTP 表挪作答案，继续追踪运行时配置选择与 framework 行为。
4. **可选体验项（P1/P2）**：LTPO 控制、PlusKey、蓝牙配对和 GNSS 首次定位，仅在取得设备证据后处理。

## 明确不做

- 不整体替换正在工作的 `.501` / `.401` proprietary blob 组合。
- 不从 `.500` 参考树移植 ABL、XBL、TZ、HYP、AOP、modem、DSP 等固件。
- 不因为参考补丁存在就屏蔽 vendor error `10108`；当前主要复现错误是 `15001`。
- 不把工作正常的相机、充电、NFC、USB 或触摸栈作为“先换了再试”的目标。

这些约束用于保护已经验证的双槽救援能力和硬件基线。任何高风险变更都必须同时具备：可复现问题、最小补丁、回滚路径和变更后设备测试。

完整的 `.501` 包、分区、指纹、Wi-Fi、相机和运营商文件级证据见 [`.501` 原厂包与现有设备树交叉审计](stock-501-cross-audit.md)。
