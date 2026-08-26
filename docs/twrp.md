# TWRP 研究与实机验证

更新时间：2026-08-26

> **2026-08-26 更新**：B 槽现为 Lineage Recovery，EDL 完整回读、启动和 root ADB 已确认；本页 TWRP 的 PASS 仅指历史环境。见 [实机报告](edl-arb1-device-validation-2026-08-26.md)。

## 结论摘要

在本机上，TWRP 的完整功能路径是把 kernelless recovery 镜像放入单侧 `recovery_a/b`，再走正常 recovery 启动流程。直接把同一 recovery 镜像交给 `fastboot boot` 会失败；给镜像补入 kernel 后，boot header v4 与 v3 又表现出不同限制。

| 路径 | 结果 | 核心原因 |
|---|---|---|
| `fastboot boot` 原始 recovery.img | FAIL：`Bad Buffer Size` | recovery header v4 的 `kernel_size=0`，downloaded-boot 校验拒绝 |
| kernel-bearing header v4 | ABL 接受，但进入 ColorOS | 存在 `init_boot` 时，ABL 使用槽内 generic ramdisk，忽略下载镜像 ramdisk |
| kernel-bearing header v3 | TWRP UI/触控启动成功 | v3 避开 `init_boot` ramdisk 替换 |
| header v3 临时启动的完整救援能力 | FAIL / 不完整 | v3 不加载 bootconfig，导致 USB/ADB、bootdevice 和挂载链缺失 |
| TWRP 写入 `recovery_b` 后正常启动 | PASS | 原厂 recovery 路径从 `boot_<slot>` 取 kernel，从 `recovery_<slot>` 取 recovery ramdisk，并保留 bootconfig |

## 上游来源与实机边界

[kmiit/twrp_device_oplus_sm87xx](https://github.com/kmiit/twrp_device_oplus_sm87xx) 的 README 将 OnePlus 13T（CN）列为支持设备，并声明 ADB、Display、Decryption、Fastbootd、Flashing、MTP、Sideload、Touch、USB OTG 和 Vibrator 可用。这些属于 `UPSTREAM_CLAIM`。

本仓库的实机验收基于 `PKX110_15.0.2.302(CN01)`、slot B 和 TWRP `3.7.1_16-OnePlus_13_T`；未测试项目不会因上游声明而自动变成 `PASS`。

## 历史非破坏性实机验收

| 项目 | 结果 |
|---|---|
| 两次正常启动 TWRP | PASS |
| 1216 × 2640 显示 | PASS |
| 触控、亮度、熄屏/唤醒、振动 | PASS |
| ADB 与 USB gadget | PASS |
| bootconfig 与 `/dev/block/bootdevice` | PASS |
| A/B 与 dynamic partition 枚举 | PASS |
| `/data` F2FS 挂载 | PASS |
| 用户 PIN / FBE 解密 | PASS |
| Internal Storage 读取 | PASS |
| MTP 枚举与只读浏览 | PASS |
| USB OTG | PASS |
| Reboot System / Recovery / Bootloader | PASS |
| MTP 写入 | NOT_TESTED |
| ADB sideload（本轮验收） | NOT_TESTED |
| Restore | SKIPPED |
| Fastbootd（本轮验收） | NOT_TESTED |

这轮测试没有执行 flash、wipe、format、install、restore 或 slot 修改。TWRP 正常运行可能产生自身日志和文件系统元数据写入，因此“非破坏性”不等于逐 bit 零写入。

## 为什么原始 recovery.img 不能直接临时启动

Stock `.302` recovery 与本次 TWRP recovery 都是 boot header v4、`kernel_size=0` 的 kernelless 镜像。这符合 OPlus 正常 recovery 启动契约，但 Qualcomm ABL 的 downloaded-boot 路径把输入当成完整 boot image，要求非零 kernel size，于是返回 `EFI_BAD_BUFFER_SIZE`，fastboot 显示：

```text
Failed to load/authenticate boot image: Bad Buffer Size
```

下载本身成功、文件大小小于 `max-download-size`，错误发生在 header 校验阶段，不是单纯容量、USB 或 AVB 问题。

## v4 与 v3 受控实验

两个候选使用完全相同的 stock `.302` kernel 和 TWRP ramdisk，只改变 header 版本：

| 项目 | v4 | v3 |
|---|---|---|
| ABL 接受 | PASS | PASS |
| 运行 kernel | `.302` stock | `.302` stock |
| 下载镜像 ramdisk 生效 | FAIL | PASS |
| Userspace | ColorOS | TWRP |
| TWRP display/touch | 未进入 | PASS |
| USB/ADB | ColorOS 下 PASS | FAIL |
| 可作为完整救援环境 | NO | NO |

v3 证明 TWRP ramdisk、kernel、显示和触控链可以工作，但 bootconfig 缺失使其不适合日常救援。这个实验不能替代正常 recovery 启动的实机验收。

## TWRP cleanup 构建

后续离线修复了三类问题：

1. 移除不断退出 127 且与 hbp5 重复注册接口的 `vendor.touch-aidl-1` 链。
2. Keymaster 版本检测在 HIDL 失败后回退到 AIDL KeyMint，使 pagani 的 KeyMint v3 能被识别。
3. 增加最小 crypto SELinux 类型和 file contexts；构建通过，0 个 neverallow 冲突，没有 broad allow 或 `dontaudit recovery`。

生成的 polish recovery 镜像只完成 `HOST_VERIFIED` 和静态验证，**没有实机刷入或运行验证**。它不能替代已知可用的旧 recovery 镜像。

## 安全边界

- 不要因 `fastboot boot` 失败就关闭 verity、关闭 verification 或尝试未知 header/cmdline 组合。
- 不要同时覆盖两个 recovery 槽；必须保留已知可启动槽和 stock/已验证镜像。
- TWRP 能解密旧 ColorOS `/data` 不代表它适合为 Lineage 创建 userdata；本机已证明 TWRP Format Data 与当前 Lineage metadata-encryption 路径不兼容。
- `fastbootd`、restore 和写入测试必须单独设计停止条件与回滚方案。
