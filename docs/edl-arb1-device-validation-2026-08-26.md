# ARB1 免授权 9008（EDL）：Linux 实机验证

测试日期：2026-08-25/26。设备：OnePlus 13T `PKX110 / pagani`，slot B，Android 16 / LineageOS 23.2；实读 `xbl_config_b` 匹配 `.501` 内容、声明 ARB `1`。主机：NixOS 26.05、Linux 7.1.10，原生 Linux `qcserial/TTY`。

Bootloader unlocked、kernel 6.6.142 为历史基线，本轮未重测；完整环境见 [设备基线](device-baseline.md)。

## 已验证结果

| 项目 | 结果 |
|---|---|
| 免在线授权进入 Firehose | 指定三文件成功加载，无需 OPLUS 售后账号或在线授权 |
| UFS / GPT | 6 个 LUN、12 份主备 GPT、139 个物理分区条目；CRC、范围与主备一致性通过 |
| 分区读取 | 已读取 `xbl_config_b`、B 槽 vbmeta 系列、Recovery 和启动分区；super 仅读取前 1 MiB 元数据 |
| B 槽 Recovery 更换 | `recovery_b` 已由 TWRP 换为 Lineage Recovery；最终 ACK、100 MiB 完整回读、SHA-256 和 AVB 校验通过 |
| 非目标区域检查 | `boot_b`、`vendor_boot_b`、`dtbo_b`、`init_boot_b`、`vbmeta_b` 及 LUN4 主备 GPT 写前/写后相同 |
| 启动 | EDL 普通重启后 Android B 启动完成；再经 ADB 进入 Lineage Recovery `23.2-20260814-UNOFFICIAL-pagani`，root ADB 可用 |

设备操作为 `DEVICE_VERIFIED`；镜像解析、哈希与 AVB 校验为 `HOST_VERIFIED`。本次只写 `recovery_b`，未写 A 槽、firmware、NV 或 userdata，未擦除、格式化或切槽。

## 已验证的 Linux 配置与流程

引导位于 [`resources/edl/arb1-sm8750/`](../resources/edl/arb1-sm8750/)，使用前核对 [三文件 SHA-256](edl-arb.md#收到的三文件材料)。

工具基于 [QSaharaServer / fh_loader @ a6dceddd](https://github.com/bm16ton/oplus12r-edl/tree/a6dceddd2ff74eabf0566ddbbb72e50d98826874)，入场顺序参考 [OPLUS EDL Tool v2 @ 72999902](https://github.com/salokrwhite/OplusEdlTool/tree/72999902af09e99c702866e0086482dc42b82bbb)。测试使用本地修订的原生 Linux `fh_loader`，未运行 Windows 程序；工具哈希见 [来源索引](../data/source-index.yaml)，补丁未随本报告发布。

1. 提前通过 `pkexec` 建立 root 监控，20 ms 轮询 `05c6:9008`，检测到设备后自动发送引导。
2. 入场顺序：Sahara image 13 programmer → digest → verify → signature → sha256init → UFS configure。
3. 本次 Linux TTY 路径的**实际 configure 使用 `ZlpAwareHost="0"`**；后续 `--skip_configure` 保留该配置。`--skip_signed_digest_ack` 只用于 signature 数据阶段。
4. sender 必须支持完整 XML 分片累积。空闲 XML 阶段每 25–30 秒串行执行只读 `getstorageinfo` 保活；监控、保活和读写共用独占端口锁。
5. 写入前重新读取 GPT、备份目标并核对镜像。写入须取得最终 `ACK rawmode=false`，再完整回读、逐字节比较并验证 AVB。
6. 重启时先在完整 ACK 边界停止保活，再执行普通 reset；进入 Recovery 使用已启动 Android 的 `adb reboot recovery`。

## 关键校验值

`recovery_b` 本次实读位置：LUN4，起始扇区 `545704`，`25600 × 4096 = 104857600` 字节。镜像来自 [2026-08-14 连贯 target-files](local-build-audit-2026-08-25.md)。这些位置只用于记录，不能替代下一次读取 GPT。

| 对象 | SHA-256 |
|---|---|
| 完整 `xbl_config_b`，409600 字节；Android 与 EDL 读回一致 | `a29081b7eb20752dee63efd21f6f984531e4560c153a09d115aab644736eda28` |
| Lineage recovery 候选、EDL 完整回读及后续 Android 校验一致 | `e136fd6549526a92e2ac1c805c80d7762418448824f4e2d9ec11209dbfa9bef7` |

## 注意事项

- ARB `1` 是已安装镜像的声明值，不是熔丝/RPMB 计数器读数；本测试不证明可以降级或绕过 ARB。
- 免在线授权不等于全分区权限。当前引导不允许 EDL 读取 `misc`；需要时可通过已授权的 Android root 备份。
- 缺少最终 ACK、raw-mode 状态不明或连接断开时立即停止；**不要插入 XML 保活或盲目重试**。
- 保留目标的双份校验备份，至少一份放在另一物理介质；不要套用旧槽位、扇区位置或混合构建产物。参见 [备份边界](backup-and-recovery.md)。
- 当前 B 槽是 Lineage Recovery，不是 TWRP。仅验证启动和 root ADB；显示与 `/data` 挂载仍有警告，触控、解密和备份恢复须另行验收。未验证 EDL 直接进入 Recovery 或其他分区写入。

原始日志与镜像已在本地留存；仓库仅发布脱敏结论、文件指纹和来源索引。
