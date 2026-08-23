# 酷安 OnePlus 13T 类原生 ROM 线索索引

更新时间：2026-08-24

本页记录酷安中可以定位到的 OnePlus 13T（`PKX110` / `pagani`）类原生 ROM 发布、更新和构建线索，并与仓库已有内容按 **ROM 家族** 排重。所有酷安内容均为 `COMMUNITY_CLAIM + NOT_TESTED`；出现下载目录、启动截图或作者自述，不等于本仓库已经验证可启动、可日用或可安全刷入。

这里把“基于 LineageOS 构建”宽松理解为“属于 pagani 类原生适配生态，可能复用 LineageOS/common device tree、hardware 或 vendor 工作”。仅凭 ROM 名称、文件目录或酷安帖子，无法证明其源码继承关系；因此下表不会把 PixelOS、Evolution X 等直接写成 LineageOS 派生项目。

## 排重后的结果

| ROM 家族 | 排重结论 | 可定位的酷安来源 | 其他发布入口 | 当前边界 |
|---|---|---|---|---|
| PixelOS | **新增家族**；首发、启动进度和 QPR2 更新合并记录 | [首发帖 `69946498`](https://www.coolapk.com/feed/69946498)、[QPR2 更新 `70455403`](https://www.coolapk.com/feed/70455403) | [SourceForge：PixelOS-13T](https://sourceforge.net/projects/oneplus-pagani/files/PixelOS-13T/) | 酷安构建与当前 SourceForge 文件是否同一维护者/源码基线，未确认 |
| DerpFest 16.2 | **仓库已有线索，补齐原帖**；不重复算作新家族 | [2026-08-03 构建 `73069893`](https://www.coolapk.com/feed/73069893)、[指纹修复更新 `72963764`](https://www.coolapk.com/feed/72963764)、[初版 `72942991`](https://www.coolapk.com/feed/72942991) | 帖内下载目标未从只读接口完整解析 | 只记录发布声明和已知问题；未下载、未刷入 |
| LineageOS 23.2 | **仓库已有且本机另有实测构建，社区帖子按不同构建流合并** | [Synecdoche 构建帖 `69337906`](https://www.coolapk.com/feed/69337906)、[好好335 近期构建帖 `73209088`](https://www.coolapk.com/feed/73209088) | [SourceForge：LineageOS-13T](https://sourceforge.net/projects/oneplus-pagani/files/LineageOS-13T/) | 三者不能视为同一个 zip、同一次构建或同一组测试结果 |
| Evolution X | **新增候选家族；未找到单独的 13T 酷安发布帖** | [酷安汇总线索 `73134391`](https://www.coolapk.com/feed/73134391) | [SourceForge：Evolution-X_13T](https://sourceforge.net/projects/oneplus-pagani/files/Evolution-X_13T/) | 只确认目录中存在 `pagani` 文件；未确认酷安原作者或安装要求 |
| MistOS | **新增候选家族；未找到单独的 13T 酷安发布帖** | [酷安汇总线索 `73134391`](https://www.coolapk.com/feed/73134391) | [SourceForge：MistOS](https://sourceforge.net/projects/oneplus-pagani/files/MistOS/) | 同上 |
| ASCP-OS | **新增候选家族；未找到单独的 13T 酷安发布帖** | [酷安汇总线索 `73134391`](https://www.coolapk.com/feed/73134391) | [SourceForge：ASCP-OS](https://sourceforge.net/projects/oneplus-pagani/files/ASCP-OS/) | 同上 |
| crDroid | **新增候选家族；未找到单独的 13T 酷安发布帖** | [酷安汇总线索 `73134391`](https://www.coolapk.com/feed/73134391) | [SourceForge：CrDroid-13T](https://sourceforge.net/projects/oneplus-pagani/files/CrDroid-13T/) | 目录同时含 recovery 与 GApps；不能把目录内所有文件都计为 ROM |

## 已定位帖子的有效信息

### PixelOS

发布账号：[蹦蹦奇趣蛋（`709398`）](https://www.coolapk.com/u/709398)。

- [2026-01-26 首发帖](https://www.coolapk.com/feed/69946498)标注 `PixelOS Unofficial 2026.01.26`，帖子自述使用 2026 年 1 月安全补丁、2025 年 12 月 vendor，以及 `PKX110_16.0.2.401(CN01)` blobs。作者称基础硬件、HDR 和密码设置可工作，同时列出偶发指纹常亮、AOD 指纹不可用和内置相机最高 1080p60 等问题。
- [2026-03-01 QPR2 更新帖](https://www.coolapk.com/feed/70455403)标注 `.501` 基础固件，并列出 AOD 指纹、首次设置后指纹、2.4 GHz Wi-Fi/热点和相机 4K 等已知问题。
- [QPR2 启动进度帖](https://www.coolapk.com/feed/70270467)仅作为上述 QPR2 更新的前置动态，不另计一个 ROM 或版本。

SourceForge 在本次检查时另有 Android 17 PixelOS 文件，但没有证据证明它与上述 2026 年 1—3 月酷安构建保持同一维护关系、签名或升级路径。

### DerpFest 16.2

发布账号：[INT16（`24635593`）](https://www.coolapk.com/u/24635593)。

- [2026-08-03 构建帖](https://www.coolapk.com/feed/73069893)称系统内置 KernelSU Next 与 SUSFS，已知问题为多出逻辑相机；首次启动后指纹可能不可用，重启后恢复。
- [2026-07-27 更新帖](https://www.coolapk.com/feed/72963764)称指纹问题已经修复，多余逻辑相机仍存在。
- [2026-07-26 初版帖](https://www.coolapk.com/feed/72942991)写明基础固件为 ColorOS `16.0.5.701`，并提到 ARB/非 ARB 引导文件。由于涉及分区刷写和 ARB 风险，本索引不转录帖子中的操作命令。
- [测试招募帖](https://www.coolapk.com/feed/72881529)是发布前动态，不另计一个 ROM。
- [“类原生 环境隐藏”原帖](https://www.coolapk.com/feed/73075885)与仓库此前收录的 DerpFest、KernelSU Next、SUSFS 和完整性/隐藏文本一致；现只补齐出处，不重复创建第二条资料。

### LineageOS 23.2

这里至少存在两条社区发布/维护线索，它们也不能与本仓库 2026-08-14 自行构建并于 2026-08-16 实机启动的产物混为一谈。

**Synecdoche（[`2779563`](https://www.coolapk.com/u/2779563)）**

- [2025-12-20 构建帖](https://www.coolapk.com/feed/69337906)称指纹、Wi-Fi、NFC 和蓝牙可用，同时记录侧键与首次设置密码延迟；后续编辑提到 HDR、麦克风音量和 Weaver 相关调整。
- [2.4 GHz Wi-Fi 修复线索](https://www.coolapk.com/feed/71274842)是该构建流的排障动态，不另计 ROM。

**好好335（[`27879362`](https://www.coolapk.com/u/27879362)）**

- [近期开发版汇总帖](https://www.coolapk.com/feed/73209088)称已集成部分一加相机功能、修复 VOOC 快充并更新安全补丁，同时保留图库预览、防抖和首次设置指纹等问题。
- [LineageOS 23.2 常规更新](https://www.coolapk.com/feed/72944538)与[“应该可以日用”构建帖](https://www.coolapk.com/feed/72884209)合并到同一构建流，不按动态数量重复计数。
- [OnePlus 13T LineageOS 编译说明](https://www.coolapk.com/feed/72954663)属于构建参考，不是另一个 ROM 包。

## 只有汇总帖和目录的候选包

[大叔累了（`2786904`）的酷安帖](https://www.coolapk.com/feed/73134391)称有国外开发者制作了多款 13T 类原生 ROM，其外部线索与仓库此前已登记的 [OnePlus Pagani SourceForge 总目录](https://sourceforge.net/projects/oneplus-pagani/files/)相符。本次在目录中确认到 PixelOS、Evolution X、MistOS、ASCP-OS、crDroid 和 LineageOS 六个 ROM 文件夹。

其中 Evolution X、MistOS、ASCP-OS 和 crDroid 暂未在本轮精确检索中找到独立的 OnePlus 13T 酷安发布帖。因此目前只能写成“**SourceForge 目录存在，酷安有集合式转链**”，不能写成“已找到酷安作者发布并验证”。

## 本轮不计入的结果

- OxygenOS 转换包、ColorOS 主题和官方系统更新：不是本页的类原生 ROM 范围。
- OrangeFox、TWRP、recovery 镜像：属于恢复环境，不是 ROM 家族。
- “求推荐”“谁有包”“准备购买”等讨论帖：没有提供可核对的发行物或维护信息。
- 其他设备或“Android 13 / 13T”文字碰撞：没有 `PKX110` / `pagani` 证据。
- 同一作者的开机进度、测试招募、修复更新和编译说明：保留链接，但只归并到对应 ROM 家族。

## 检索与链接边界

本轮使用酷安公开只读接口，围绕 `一加13T`、`PKX110`、`pagani` 与各 ROM 名称组合检索，并参考公开的 [qiuyurs/coolApkAPI](https://github.com/qiuyurs/coolApkAPI) 请求实现。仓库只保存稳定的 `https://www.coolapk.com/feed/<id>` 链接，不保存搜索结果中的临时分享参数。

帖子详情/评论接口在本轮后段返回异常，部分“查看链接”、网盘地址或访问码无法完整解析。本页不会猜测缺失目标；外部下载只登记能够独立打开和核对的 SourceForge 目录。后续如补到原帖评论中的下载地址，应作为新证据单独提交，并重新检查文件哈希、基础固件、ARB 与回滚说明。
