# OPlus / ColorOS 相机移植线索

更新时间：2026-08-24

> **状态：`SOURCE_VERIFIED + STOCK_FILES_HOST_VERIFIED + COMMUNITY_CLUE + NOT_TESTED_ON_LINEAGE`**
>
> 这条记录讨论的是把 OPlus/ColorOS 原厂相机能力移植到其他系统，不影响当前 LineageOS 已验证的基础 Camera HAL 和普通相机功能 `PASS`。

## 直接源码线索

仓库 [`kinginu/oneplus13t-fix-camera`](https://github.com/kinginu/oneplus13t-fix-camera/tree/698cdca07efc12f9d5c8d3ad134e011289bcabd0) 面向“国行 OnePlus 13T 转 OxygenOS/OnePlus 13s”场景。README 声称通过使用 ColorOS 的 ODM 内容修复相机和电池容量；当前核对提交为 `698cdca07efc12f9d5c8d3ad134e011289bcabd0`。

源码显示它不是简单替换一个 APK，而是在启动时 bind-mount 多组 ColorOS ODM 文件：

- `/odm/etc/camera/` 下的相机配置、算法开关、DNG、EIS、传感器和 framework config；
- `/odm/lib64/camera/` 下的 QTI/OPlus 调校文件与算法插件；
- `/odm/lib64/hw/camera.oemlayer.so`；
- 同时包含 power profile、fast-charge 配置等非相机 ODM 内容。

项目还记录了 KernelSU/APatch 开启 **Umount modules** 时，原厂相机多帧后处理可能看不到模块文件，造成拍照后一秒变黑；这只证明模块覆盖层必须对相机进程可见，不等于 LineageOS 已具备完整的 OPlus framework。

## `.501` 文件级对照结果

本轮使用与当前救援槽版本相同的公开 `PKX110_16.0.3.501(CN01)` 完整 OTA，只做主机侧提取和哈希比较：

- 仓库的 `CPH2723_16.0.3.501` 标签固定到 [`076bf52e7d9b88b5620f83efe7276847cb20a18c`](https://github.com/kinginu/oneplus13t-fix-camera/tree/076bf52e7d9b88b5620f83efe7276847cb20a18c)；
- 该提交 `module/odm` 下共 101 个文件，在 PKX110 `.501` ODM 中 101/101 存在，且 101/101 SHA-256 相同；
- 冻结 pagani `proprietary-files.txt` 中 916 个非注释相机相关条目，在本次提取分区中 916/916 路径存在。

因此“先把模块文件与当前清单排重”的主机工作已经完成：没有发现需要整批补入的相机文件集合。这个结果证明的是 `.501` 文件来源和清单覆盖，不证明 OPlus framework、HDR、夜景、变焦或多帧处理在 LineageOS 中完整工作。完整包与分区证据见 [`.501` 交叉审计](stock-501-cross-audit.md#相机)。

## 社区截图中的两条判断

用户提供的 CoolApk 截图显示：

1. **“爱干饭的圣徒”**：相机移植的关键在 ODM 分区文件，可参考上述模块；该模块用于在 OnePlus 13T 的 OxygenOS 上恢复国行相机功能。
2. **“好好335”**：LineageOS 已提取其中大部分 ColorOS 文件，真正影响原厂相机移植的可能是 OPlus 庞大的 framework；缺少 framework 时相机会黑屏。

截图未显示精确帖子 URL 和年份，因此只作为社区上下文，不作为源码事实。

## 当前判断

这两条说法并不完全冲突：

- 在 OxygenOS 转换场景中，OxygenOS 已有大量 OPlus framework/service，补齐 PKX110 对应的 ODM 配置和算法库可能足以恢复相机。
- 在 LineageOS 场景中，ODM blob 即使齐全，也可能仍缺 OPlus framework 类、system service、权限、SELinux 或 ABI 依赖，因此不能直接套用这个模块。

合理的下一步是保留当前已经工作的栈，用实机矩阵分别测试基础拍摄、HDR、2.1× 以上变焦、前后摄切换、夜景和长时间录像；只有复现具体功能缺口后，才结合 `com.oplus.camera` 日志定位 framework 类、service、权限、SELinux 或 ABI 依赖。不要把整个模块直接挂载到当前 LineageOS，也不要把其公开 Release 当作适配完成证明。
