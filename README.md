# FanchmWrt

FanchmWrt 是一款开源的企业级路由器系统，基于 OpenWrt 深度定制，内置了完整的防火墙与网络管理能力。

多款主流型号固件已发布，可在 [www.fanchmwrt.com](https://www.fanchmwrt.com) 下载；你也可以按本文档自行编译其它型号的固件。

## 主要特性

本项目在 OpenWrt 基础上加入了以下核心能力（相关源码见 `package/fcm`）：

- **应用识别与过滤（OAF）**
  通过内置的应用特征库（`feature.bin`，来源于 OAF 应用特征文件）对网络流量做应用协议识别，实现按应用维度的识别、管控与过滤。
- **MAC 地址过滤**
  支持基于 MAC 地址的接入控制与过滤规则。
- **FullCone NAT（NAT1）**
  内置 `fullconenat` / `fullconenat-nft`，在 nftables 框架下实现全锥形 NAT，改善 P2P、游戏等场景的连通性。
- **无线与网络管理**
  提供无线接入、设备/用户管理、流量统计与行为记录等功能（`fwxd` 守护进程 + `libfwx_common`）。
- **本地数据与远程上报**
  使用 SQLite 本地持久化记录，并通过 MQTT（mosquitto）等通道支持数据上报。
- **OpenWrt 生态深度集成**
  通过 ubus / rpcd 与 LuCI 无缝对接，并提供定制化 LuCI 主题（`luci-theme-fanchmwrt`）与界面增强补丁（`feeds_patches/luci`）。

## 重要说明

- 本项目对个人使用免费；允许再分发固件或将代码移植到其它项目，但所有源代码文件中的版权信息必须保留。
- OAF 应用特征文件用于描述应用协议特征，个人可免费使用，**禁止商用**。你可以自行提取应用特征，但不得直接使用开源的特征文件；这些文件的版权属于 FanchmWrt。

## 开发与编译

编译 FanchmWrt 的方式与 OpenWrt 相同，推荐使用 Ubuntu 22。重新发布固件时请注明本仓库地址。

### 环境要求

编译 FanchmWrt 与 OpenWrt 所需的工具相同，包名随发行版而异。各发行版的完整依赖清单见
[Build System Setup](https://openwrt.org/docs/guide-developer/build-system/install-buildsystem) 文档。

```
binutils bzip2 diff find flex gawk gcc-6+ getopt grep install libc-dev libz-dev
make4.1+ perl python3.7+ rsync subversion unzip which
```

### 快速开始

编译 FanchmWrt 与编译 OpenWrt 完全一致，可参考 OpenWrt 编译教程。

1. 运行 `./scripts/feeds update -a` 获取 feeds.conf / feeds.conf.default 中定义的全部最新软件包。
2. 运行 `./scripts/feeds install -a` 将获取到的软件包符号链接到 `package/feeds/`。
3. 运行 `make menuconfig` 选择工具链、目标系统与固件软件包。
4. 运行 `make` 开始编译：下载全部源码、构建交叉编译工具链，然后交叉编译内核与所选应用，最终生成固件。

### 编译 ARMv8 虚拟机 / EFI 固件

仓库内置 GitHub Actions 工作流 **"Build ARMv8 VM + EFI firmware"**（`build-armv8-vm.yml`），可在 Actions 页面手动触发，或推送 `build-*` 前缀的 tag 触发。构建目标为 OpenWrt `armsr/armv8`（Generic EFI Boot），产物包括：

- `squashfs-rootfs.img.gz` —— armv8（aarch64）虚拟机可直接使用的 squashfs 根文件系统镜像；
- `squashfs-combined-efi.img.gz` —— 含 EFI 引导的整盘镜像（combined EFI 固件）；
- `sha256sums` —— 镜像校验值。

每次运行会把产物发布到同名仓库的 Release：`fanchmwrt-armv8-vm-latest`。

## 上游仓库

https://github.com/openwrt/openwrt

## 许可证

FanchmWrt 采用 GPL-2.0 许可证。
