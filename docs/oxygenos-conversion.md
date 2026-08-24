# PKX110 转 CPH2723 OxygenOS 社区流程参考

更新时间：2026-08-24

> **当前状态：`COMMUNITY_GUIDE + SOURCE_PARTIALLY_VERIFIED + NOT_TESTED`**
> 本页收录 XDA 转换主帖中的可复用信息、明确链接和风险边界，不是刷机教程。本仓库没有下载、重传或执行论坛中的 flasher、镜像、模块及网盘包。

## 来源入口

| 内容 | 明确链接 | 用途与状态 |
|---|---|---|
| XDA 主帖 | [PKX110 OxygenOS Convert — OnePlus 13T to 13s](https://xdaforums.com/t/rom-pkx110-oxygenos-convert-oneplus-13t-to-13s.4743853/)／[首帖固定链接](https://xdaforums.com/t/rom-pkx110-oxygenos-convert-oneplus-13t-to-13s.4743853/post-90142594) | 社区转换流程与限制；未实机复测 |
| 分区作用解释 | [XDA post 90143442](https://xdaforums.com/t/rom-pkx110-oxygenos-convert-oneplus-13t-to-13s.4743853/post-90143442) | 社区作者对 `odm`、`my_region`、`oplusstanvbk` 的用途说明 |
| OTA 覆盖说明 | [XDA post 90144680](https://xdaforums.com/t/rom-pkx110-oxygenos-convert-oneplus-13t-to-13s.4743853/post-90144680) | 更新后信号／相机修复可能被覆盖 |
| 文件镜像入口 | [Daniel Springer：OxygenOS Flashers](https://roms.danielspringer.at/index.php?dir=Oneplus+13T%2FOxygenOS+Flashers) | 论坛提供的镜像；未下载、未验哈希、未审计 |
| 信号模块 | [fix-signal-oneplus13t-v3.1.zip](https://xdaforums.com/attachments/fix-signal-oneplus13t-v3-1-zip.6237828/) | XDA 附件；仅登记，不重传、不背书 |
| 相机模块源码 | [`kinginu/oneplus13t-fix-camera`](https://github.com/kinginu/oneplus13t-fix-camera/tree/698cdca07efc12f9d5c8d3ad134e011289bcabd0) | 已做源码级清单检查；详见[相机移植线索](camera-porting-clue.md) |
| 后续相机模块版本 | [XDA post 90525954](https://xdaforums.com/t/rom-pkx110-oxygenos-convert-oneplus-13t-to-13s.4743853/post-90525954)／[CPH2723_16.0.5.700 release](https://github.com/kinginu/oneplus13t-fix-camera/releases/tag/CPH2723_16.0.5.700) | 上游称该版本取材于 `PKX110_16.0.5.701`；未在本仓库设备上测试 |

## 流程结构，而非操作步骤

主帖描述的是把国行 `PKX110` 的 ColorOS 用户空间转换为 `CPH2723` OxygenOS。其核心并非只写入一个 system image，而是保留或重新叠加 13T 所需的区域、蜂窝和相机差异。

| 路线 | 社区流程所依赖的修复层 | 已知维护成本 |
|---|---|---|
| Root 路线 | 在 OxygenOS 基础上加载信号和相机模块 | 更新后需要核对模块与新版本兼容性；root 本身引入完整性检测与安全边界 |
| Non-root 路线 | 按主帖组合 ColorOS `oplusstanvbk.img`、`odm.img` 与 OPPO 来源的 `my_region.img` | OTA／本地更新可能覆盖相关分区，随后信号或相机再次失效，需要重新处理 |

上述是论坛作者的流程描述，不是本仓库对分区组合安全性的确认。尤其不能把其中镜像与任意固件代际混用，也不能据此绕过当前设备的 ARB 边界。

## 社区给出的分区角色

- `odm.img`：作者称主要用于补回 13T 与 13s 不同的相机驱动、配置和算法依赖。
- `my_region.img`：作者称取自全球版 OPLUS 设备，用于 Google 服务及区域功能。
- `oplusstanvbk.img` 与 `my_region.img`：作者称两者组合与蜂窝信号恢复有关。

这些角色说明与 [`oneplus13t-fix-camera`](https://github.com/kinginu/oneplus13t-fix-camera/tree/698cdca07efc12f9d5c8d3ad134e011289bcabd0) 的 ODM bind-mount 清单方向一致，但仍不足以证明整套转换的来源完整性、版本兼容性或可恢复性。

## 已记录的限制与风险

- 主帖称 Mind Space 的连接功能不可用，尚无已知修复。
- root 与 non-root 路线都可能在 OTA 后需要重新核对信号和相机；不能把“能够更新”写成“更新后无需维护”。
- [一条用户报告](https://xdaforums.com/t/rom-pkx110-oxygenos-convert-oneplus-13t-to-13s.4743853/post-90553867)描述转换后黑屏并进入 EDL；[作者追问版本并建议 EDL 救援](https://xdaforums.com/t/rom-pkx110-oxygenos-convert-oneplus-13t-to-13s.4743853/post-90553914)，报告者随后说明是从 `15.0.2.501` 转到 `16.0.5.700`（[post 90553966](https://xdaforums.com/t/rom-pkx110-oxygenos-convert-oneplus-13t-to-13s.4743853/post-90553966)）。这只是单例风险记录，不能据此断定直接因果或所有版本都会失败。
- 转换包、镜像和模块的许可证、签名、哈希与供应链尚未统一审计；本仓库不重新分发。

## 后续采用前必须补齐

1. 固定转换前 ColorOS、目标 OxygenOS、当前槽位和 rollback index。
2. 为每个外部文件记录来源 URL、大小与 SHA-256，并确认它对应 `PKX110`／`CPH2723` 的具体版本。
3. 分清 boot-critical、firmware、逻辑分区与 root bind-mount；任何低 ARB 启动组件都不得写入当前 ARB 1 基线。
4. 预先准备与同代固件匹配的恢复路径，再分别测试蜂窝、相机、OTA、槽位切换和重启。

在这些项目完成前，本页仅是该转换主帖的社区资料索引，不是推荐方案。
