# Henge 汉化 + 多应用转译增强（非官方分支）

在 [Leviidev/Henge](https://github.com/Leviidev/Henge) 之上做的一组改动：**简体中文界面**、**多个 Mac 应用同屏运行**、**动态分辨率**、**应用可见的机型身份**。

> 本仓库是衍生作品，原项目版权归 Leviidev 及贡献者所有，按 **GPL-2.0-or-later** 发布。
> 原项目：https://github.com/Leviidev/Henge

## 它做什么

Henge 在 iPhone / iPad 上运行 macOS 应用，**不需要启动 macOS 虚拟机**：它自带一个 Mach-O 加载器，把 macOS 的 arm64 程序载入 Henge 进程，把程序对系统库的调用接到 iOS 自己的同名库上，缺失的 AppKit 由 Henge 自己实现的一套替代层补上。

本分支在这个基础上：

| 改动 | 说明 |
|---|---|
| **界面全中文** | 首页、设置、Mac 应用、桌面、会话、安装、关机页全部简体中文 |
| **多应用同屏** | 每个应用一个 loader namespace，共用一块屏幕和一条菜单栏；点窗口即切前台 |
| **动态分辨率** | `min(容器尺寸, 面板像素/缩放) × 密度`，跟随窗口变化实时重算，永不超过设备面板 |
| **机型身份** | 应用看到的 `hw.model` / 芯片 / 内存可配置（默认 iPhone OS / Apple A12Z / 4 GB） |

## 不是什么

- **不是 GPU 驱动**。Henge 不需要写显卡驱动：macOS 的图形调用由半虚拟 GPU 转成 Metal 回放（`devices/pvg_metal.m`）。苹果从未开源 macOS 的 GPU 驱动。
- **不是指令翻译**。M 系列与 A 系列同为 ARM64，指令集相同，没有指令要"翻译"；要翻译的是 API 层。真正需要指令翻译的只有 x86_64 的 Mac 程序（Rosetta 场景），本项目不支持。
- **不能跑在 iPhone 6S 上**。需要 iOS 17 及以上的 JIT 路径，6S 最高 iOS 15.8。

## 构建

需要 Mac + Xcode 26 以上 + xcodegen。步骤与原项目一致：

```sh
cd ios && xcodegen
make ios-lib
ios/scripts/fetch_deps.sh
tools/build_ios_restore.sh
ios/scripts/package_ipa.sh
```

## 已知限制

- 多个应用各带一份 Foundation / Electron Framework 时会撞同名类，行为未定义
- 共用一块最大约 1.5 GB 的可执行区域
- 应用自己生成可执行内存（V8 JIT）仍然不行，Chromium 系需要 `--jitless`
- 一次启动后不能再开 macOS 虚拟机

## 许可

GPL-2.0-or-later。见 `LICENSE` 与 `CREDITS.md`。
