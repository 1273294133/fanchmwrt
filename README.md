# FanchmWrt

## 仓库说明（中文）

FanchmWrt 是一款基于 OpenWrt 深度定制的开源企业级路由器系统，内置防火墙与网络管理能力。

<div align="center">
<h3 style="color:#1f6feb; font-size:1.6em; border-bottom:3px solid #1f6feb; padding-bottom:6px; margin-bottom:6px;">
本仓库可用于生成可运行于 ARMv8 虚拟机的固件
</h3>
</div>

- 通过内置的 GitHub Actions 工作流 **“Build ARMv8 VM + EFI firmware”**（手动触发，或推送 `build-*` 前缀的 tag），即可编译并自动发布 ARMv8（aarch64）虚拟机可运行的固件；
- 产物包括 `squashfs-rootfs.img.gz`（ARMv8 虚拟机可直接使用的 squashfs 根文件系统镜像）与 `squashfs-combined-efi.img.gz`（含 EFI 引导的整盘固件），发布到仓库 Release：`fanchmwrt-armv8-vm-latest`；
- 构建目标为 OpenWrt `armsr/armv8`（Generic EFI Boot），编译完成后即可用 QEMU 等 ARMv8 虚拟机直接启动。

#### ARMv8 虚拟机使用（QEMU）

两个产物的用途与启动方式：

- **`*squashfs-combined-efi.img.gz`**：整盘固件（含 UEFI 引导、内核与根文件系统），可直接作为 armv8 虚拟机的磁盘镜像启动，日常 ARMv8 虚拟机使用推荐它。
- **`*squashfs-rootfs.img.gz`**：squashfs 根文件系统镜像，不含引导与内核，适合作为根分区挂载，或自行加载内核后作为 root 使用。

以 QEMU（aarch64）启动 combined-efi 整盘镜像为例（先解压 `.img.gz`）：

```bash
# 1. 解压镜像
gzip -dk openwrt-armsr-armv8-generic-squashfs-combined-efi.img.gz

# 2. 使用 UEFI 固件（AAVMF/QEMU_EFI）启动
qemu-system-aarch64 \
  -M virt -cpu cortex-a57 -m 1024 -smp 2 \
  -bios /usr/share/AAVMF/AAVMF_CODE.fd \
  -drive file=openwrt-armsr-armv8-generic-squashfs-combined-efi.img,format=raw,if=virtio \
  -netdev user,id=lan -device virtio-net-pci,netdev=lan \
  -nographic
```

> 提示：不同发行版 UEFI 固件路径不同（Ubuntu/Debian 为 `/usr/share/AAVMF/AAVMF_CODE.fd`，也可用 QEMU 自带的 `QEMU_EFI.fd`）。rootfs 镜像不含引导，如需使用可先挂载查看，或与内核、引导组装成可启动镜像。

此外，本仓库还内置以下路由器特性：

- **应用识别与过滤（OAF）**：基于内置应用特征库（`feature.bin`）对流量做应用协议识别与管控。
- **MAC 地址过滤**：基于 MAC 的接入控制与过滤规则。
- **FullCone NAT（NAT1）**：内置 `fullconenat` / `fullconenat-nft`，基于 nftables 实现，改善 P2P、游戏等场景连通性。
- **无线与网络管理**：无线接入、设备/用户管理、流量统计与行为记录（`fwxd` 守护进程 + `libfwx_common`）。
- **本地存储与远程上报**：SQLite 本地持久化，MQTT（mosquitto）数据上报。
- **OpenWrt 生态集成**：与 ubus / rpcd / LuCI 深度对接，含定制 LuCI 主题（`luci-theme-fanchmwrt`）与界面增强补丁（`feeds_patches/luci`）。

官方固件下载见 [www.fanchmwrt.com](https://www.fanchmwrt.com)。

---

## About
FanchmWrt is an open-source enterprise-grade router system.  
This project is based on OpenWrt and incorporates some firewall features.    
Several popular device firmwares have been released and are available for download at [www.fanchmwrt.com](https://www.fanchmwrt.com).   
You can also compile firmwares for other models yourself.

## Important Notes  
This project is free for personal use, you may redistribute the firmware or port the code to other projects, however,the copyright information in all source code files must be retained.  
The App feature file of OAF is used to describe the protocol characteristics of an app, individuals may use it for free, but commercial use is prohibited,you can extract application characteristics yourself but not directly use the open-source feature files, the copyright for these files belongs to FanchmWrt.    

## Development
You can compile the fanchmwrt firmware yourself, Ubuntu 22 is recommended.
Please include the repository address when re-releasing firmware.

### Requirements
You need the following tools to compile FanchmWrt the same as OpenWrt, the package names vary between
distributions. A complete list with distribution specific packages is found in
the [Build System Setup](https://openwrt.org/docs/guide-developer/build-system/install-buildsystem)
documentation.

```
binutils bzip2 diff find flex gawk gcc-6+ getopt grep install libc-dev libz-dev
make4.1+ perl python3.7+ rsync subversion unzip which
```

### Quickstart
Compiling FanchmWrt is the same as compiling OpenWrt; please refer to the OpenWrt compilation tutorial.

1. Run `./scripts/feeds update -a` to obtain all the latest package definitions
   defined in feeds.conf / feeds.conf.default

2. Run `./scripts/feeds install -a` to install symlinks for all obtained
   packages into package/feeds/

3. Run `make menuconfig` to select your preferred configuration for the
   toolchain, target system & firmware packages.

4. Run `make` to build your firmware. This will download all sources, build the
   cross-compile toolchain and then cross-compile the GNU/Linux kernel & all chosen
   applications for your target system.

## License

FanchmWrt is licensed under GPL-2.0

## Upstream Repository
https://github.com/openwrt/openwrt


