# EDL、Firehose 与 ARB1 待验证引导

更新时间：2026-08-24

> **当前状态：`COMMUNITY_CLAIM + HOST_VERIFIED_STRUCTURE + NOT_TESTED`**
>
> 本页记录一个待验证救援线索，不表示该组三文件已经在 PKX110 上成功进入 Firehose，更不表示可以安全降级或刷写。原压缩包及其中二进制不会进入仓库。

## 来源与例外范围

本资料由仓库所有者归属给 CoolApk 用户 [“皓皓的小月”](https://www.coolapk.com/u/39276643)（用户 ID `39276643`），主题范围仅限 OnePlus 13T 的 ARB1 免授权 EDL/Firehose 引导。此次没有提供精确帖子 URL，且审计环境无法直接读取该个人主页，因此“账号—资料”的归属目前仍以仓库所有者提供的信息为依据。

这是仓库对默认“排除 CoolApk 来源”规则设立的限定例外之一。例外仅允许保留归属、静态检查结果和待验证步骤，不扩展到该账号其他内容，也不把社区说法提升为实机事实。

## 先区分四个概念

| 概念 | 负责什么 | 不能据此推出什么 |
|---|---|---|
| EDL / Qualcomm 9008 | SoC 的紧急下载入口；主机通常看到 Qualcomm 9008 USB 设备 | 看到端口不代表 programmer 已通过认证 |
| Sahara | EDL 初始传输阶段，用来协商并加载 programmer | 成功发送文件不一定已经取得存储访问 |
| Firehose | programmer 运行后的命令协议，可提供分区枚举、读取、写入和擦除能力 | 能进入 Firehose 不代表旧启动链可以通过 ARB |
| ARB / rollback protection | 记录安全版本，并拒绝启动 rollback index 更低的组件 | ARB 1 不等于 9008 端口被物理关闭，也不等于所有 Firehose 都必然失效 |

Android 官方对 rollback protection 的定义是：在防篡改存储中记录版本，并拒绝启动低于已记录版本的 Android 组件；具体版本通常按分区跟踪。由此可知，“能否进入 Firehose”和“旧引导能否启动”处在不同检查层。

社区所称“免授权 9008”，更准确地说是：在不登录 OPLUS 官方售后账号的情况下，使用一组可被设备接受的 programmer、digest 和 signature，从 Sahara 继续进入 Firehose。它不表示运行了任意未签名代码，也不自动绕过 ARB。

## 社区时间线

- **2025-11-27 左右**：XDA 出现 [OPPO/OnePlus/Realme Qualcomm Files Share](https://xdaforums.com/t/oppo-oneplus-realme-qualcomm-files-share.4769736/)。其后社区开始共享部分 OPLUS 高通机型的 programmer、digest 和 signature，并配合 EDL 工具尝试在无售后账号的情况下进入 Firehose。
- **2026 年初**：社区报告逐渐出现两类变化：旧的公开 loader/流程在新固件上被拒绝或不再兼容；部分机型的新固件开始提升 ARB。没有找到将两者统一描述为同一机制的官方公告，因此这里只把它们记录为时间上相近、技术上独立的变化。
- **OnePlus 13T 边界**：在 [OnePlus Anti-Rollback Checker](https://github.com/Bartixxx32/Oneplus-antirollchecker/tree/6088e36feb702925e21978aae2c3e030d148cfca) 的 2026-08-23 数据中，`PKX110_16.0.3.501(CN01)` 为 ARB `1`，`PKX110_16.0.2.400(CN01)` 为 ARB `0`。本仓库没有从 `.501` OTA 独立复算该值，因此将其记为公开来源交叉证据。

“ARB 熔断后无法通过 9008 深刷”不是准确表述。更新后 EDL/9008 入口可能仍存在；失效的可能是特定公开 programmer、签名材料或工具流程。官方售后仍可能使用受授权的工具，未来也可能出现兼容新版本的公开材料，但在实际验证前不能假定任何一组 loader 可用。

### XDA 社区 ARB 警告与证据边界

2026-01-18 的 XDA 帖 [Critical Warning: ColorOS 16.0.8.xxx / 16.0.3.501 Permanent Anti-Rollback](https://xdaforums.com/t/critical-warning-coloros-16-0-8-xxx-16-0-3-501-permanent-anti-rollback-arb-fuse-blown-do-not-update.4775930/)（[首帖固定链接](https://xdaforums.com/t/critical-warning-coloros-16-0-8-xxx-16-0-3-501-permanent-anti-rollback-arb-fuse-blown-do-not-update.4775930/post-90451800)）将 OnePlus 13／13T 的 `16.0.3.501` 列入 ARB 风险警告；[机型列表回复](https://xdaforums.com/t/critical-warning-coloros-16-0-8-xxx-16-0-3-501-permanent-anti-rollback-arb-fuse-blown-do-not-update.4775930/post-90453123)再次把 13／13T `.501` 标为已确认。

这与 Anti-Rollback Checker 对 PKX110 `.501 = ARB 1`、`.400 = ARB 0` 的版本边界相互印证，因此收录为 `COMMUNITY_WARNING + CROSS_SOURCE`。但该帖关于“永久熔丝”“旧 ROM／自定义 ROM 必然硬砖”以及“旧免授权 EDL 工具全部失效”的更强表述，没有在本仓库通过设备寄存器、OTA 元数据、programmer 握手或官方资料独立证明，不能替换本页对 EDL、Firehose 和 ARB 三层机制的区分。

## 收到的三文件材料

原始归档 `arb1免授权9008引导.zip` 大小为 732,455 bytes，SHA-256：

```text
84b3354e31aefed4caa8832f09dbca726fb015e4ae50ddfd5e40694565799d0a
```

归档完整性测试通过，无加密条目和路径穿越；只包含以下三个文件：

| 角色 | 文件 | 大小 | SHA-256 |
|---|---|---:|---|
| Programmer 候选 | `devprg.elf` | 1,682,200 | `e7e9d768175aeacb5f33f1e019121730f19faccf91ea7fd249d8c4826dfec77e` |
| Digest 候选 | `digest.elf` | 33,244 | `c4d39f9e60d5681a5b39300844e44b2f25a394566ade95e62ab413cdc8151bfe` |
| Signature 候选 | `sig.bin` | 4,096 | `e0ca7caa23ae93302b5fc0fef06a5ecd31a619f192637573ab1b2dbed840f127` |

静态检查能确认 `devprg.elf` 含 OPLUS SM8750/Qualcomm 标识，三文件的角色也与公开 [OPLUS EDL Tool v2](https://github.com/salokrwhite/OplusEdlTool/tree/72999902af09e99c702866e0086482dc42b82bbb) 所需的 programmer、digest、signature 流程相符。

静态检查**不能证明**这些文件的签名链会被 PKX110 `.501` 接受、能进入 Firehose、能正确识别 UFS/LUN，或不会触发其他安全状态；也没有证明文件的原始出处、再分发许可或供应链完整性。因此仓库只公开指纹和观察结果，不重新分发文件。

## 待验证用法

第一次只验证能否进入 Firehose，不做深刷：

1. 确认设备为 `PKX110`、记录完整固件和槽位，并核对三个文件的 SHA-256。
2. 进入 9008 后，在工具中分别选择 `devprg.elf`、`digest.elf`、`sig.bin`，只执行 **Enter Firehose**。
3. Sahara 加载、Firehose 握手及一次只读 GPT/分区表枚举均成功，才能记为可用。
4. 不执行 write、erase、rawprogram、OFP/OPS 或 ROM flash；保存脱敏日志后退出并确认双槽状态不变。

### 立即停止条件

- programmer、digest、signature 或 authorization 被拒绝；不要通过随机换文件反复尝试。
- 工具报告存储几何、UFS LUN、sector size 或目标平台不匹配。
- 只能进入端口但 Firehose 命令无响应或会话反复断开。
- 任何界面准备写入/擦除 `xbl`、`abl`、`tz`、`hyp`、`aop`、`modem`、NV、`persist`、`frp` 或 LUN5。
- 任何方案要求把 `.302` / `.400` 的 ARB 0 启动组件写入当前 `.501` ARB 1 基线。

当前所有设备侧项目仍为 `NOT_TESTED`。在 Sahara、Firehose、只读分区表和退出后启动状态均回填前，本资料不得简称为“已验证可用的 ARB1 免授权 9008”。
