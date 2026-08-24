# OPlus / ColorOS 相机移植线索

更新时间：2026-08-23

> **状态：`SOURCE_VERIFIED + COMMUNITY_CLUE + NOT_TESTED_ON_LINEAGE`**
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

## 社区截图中的两条判断

用户提供的 CoolApk 截图显示：

1. **“爱干饭的圣徒”**：相机移植的关键在 ODM 分区文件，可参考上述模块；该模块用于在 OnePlus 13T 的 OxygenOS 上恢复国行相机功能。
2. **“好好335”**：LineageOS 已提取其中大部分 ColorOS 文件，真正影响原厂相机移植的可能是 OPlus 庞大的 framework；缺少 framework 时相机会黑屏。

截图未显示精确帖子 URL 和年份，因此只作为社区上下文，不作为源码事实。

## 当前判断

这两条说法并不完全冲突：

- 在 OxygenOS 转换场景中，OxygenOS 已有大量 OPlus framework/service，补齐 PKX110 对应的 ODM 配置和算法库可能足以恢复相机。
- 在 LineageOS 场景中，ODM blob 即使齐全，也可能仍缺 OPlus framework 类、system service、权限、SELinux 或 ABI 依赖，因此不能直接套用这个模块。

合理的下一步是把模块中的 ODM 路径与当前 `proprietary-files.txt` / `vendor/oneplus/pagani` 做清单差异，再结合 `com.oplus.camera` 启动日志定位缺少的 framework 类或服务。不要把整个模块直接挂载到当前 LineageOS，也不要把其公开 Release 当作适配完成证明。
