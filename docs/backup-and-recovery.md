# 备份与恢复边界

更新时间：2026-08-23

## 已完成的备份基线

2026-08-15 在 ColorOS `PKX110_15.0.2.302(CN01)`、slot B、bootloader unlocked 的状态下完成分层备份。最终校验脚本报告所有阶段通过，SHA256 manifest 为 276/276 files、0 failures。

| 资产 | 结果 | 说明 |
|---|---|---|
| `.302` Full OTA 与 14 个提取镜像 | PASS | 签名、hash 与统一校验清单通过 |
| Boot-critical A/B | PASS | boot、init_boot、vendor_boot、recovery、dtbo、vbmeta 系列 |
| Firmware A/B | PASS | 包含 modem、dsp、vm-bootsys 等物理分区 |
| NV / calibration | PASS | persist、modemst、fsg/fsc 等；默认禁止恢复 |
| Raw super | PASS | 物理大小与 LP metadata 检查通过 |
| TWRP private data | PASS | 文件级 MD5 与 gzip 全流校验通过 |
| Internal Storage | PASS | 分卷归档与源端条目核对通过 |
| Raw userdata + metadata | PASS | size 与 SHA256 通过；只作最后手段 |

备份文件包含大量个人数据，不在本公开仓库中提供。

## 三层恢复模型

| Tier | 用途 | 原则 |
|---|---|---|
| 1 | 用户文件与应用数据 | 优先使用文件级/TWRP 备份恢复 |
| 2 | 正常 ROM rollback | 先恢复可启动的 system/boot 环境，不碰 NV |
| 3 | 设备级灾难恢复 | raw userdata+metadata 或设备专属分区；只在明确诊断后使用 |

## 最小恢复原则

- Recovery 损坏时，只处理 recovery 分区。
- Boot chain 损坏时，先处理目标槽的 boot/init_boot/vendor_boot/dtbo/vbmeta 链。
- ROM 无法启动但 `/data` 可能仍完好时，不要先恢复 userdata。
- Dynamic partition 损坏时优先使用受控的官方 OTA/兼容安装路径；raw super 不是第一选择。
- `persist`、`modemst1/2`、`fsg`、`fsc`、`frp`、keystore、uefivarstore 等备份存在，但正常 ROM 回滚不得恢复。
- Raw userdata 与 metadata 是同一代快照对，不能拆开“试试看”。

## 不能保证恢复的内容

即使文件和分区都完整，仍可能无法恢复：

- hardware-backed Keystore / StrongBox / TEE 绑定密钥；
- DRM L1 状态；
- 银行应用、2FA token 和设备绑定登录态；
- 在 factory reset、锁屏凭据变化或 firmware/rollback 状态变化后失效的加密材料。

## Lineage 实验后的新增边界

原备份时的警告“保持 PIN 不变有助于 FBE 恢复”只适用于当时仍保留原 ColorOS 数据的阶段。2026-08-16 已经明确授权并执行 userdata wipe，后续 Lineage `/data` 是新 generation；不能再把旧的 raw userdata/metadata 恢复步骤当作普通回滚。

任何真实恢复都必须根据当时槽位、firmware、metadata-encryption、bootloader 和安全状态重新设计。本页是资产与风险索引，不是可直接复制执行的命令清单。

