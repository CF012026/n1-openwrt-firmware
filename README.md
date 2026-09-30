# N1 私人 OpenWrt 固件（斐讯 N1 / Amlogic S905D）

基于 [ImmortalWrt](https://github.com/immortalwrt/immortalwrt) `openwrt-25.12` 分支（6.12 LTS 内核）自编译的**斐讯 N1 盒子固件**，插件集与本人的 360T7 / x86 固件完全一致。默认配置干净（不预置 lucky / easytier 私人数据），LAN 默认 **192.168.99.1**。

## 固件特性

| 项目 | 说明 |
| --- | --- |
| 设备 | 斐讯 N1（Amlogic S905D，aarch64） |
| LAN 地址 | `192.168.99.1`（默认，首次启动自动设置） |
| 软件空间 | 2 GB（`TARGET_ROOTFS_PARTSIZE=2048`，eMMC 8G 富余） |
| 代理后端 | daed（eBPF 透明代理；运行内核 BTF 已核实开启） |
| daed 可视化 | luci-app-daed（内嵌 2023 面板，含连接/流量统计，烤入固件） |
| 组网 | EasyTier + WireGuard + Tailscale |
| 穿透 / DDNS | Lucky 大吉 |
| 软件中心 | iStore + QuickStart + iStoreX + Linkease + ddnsto + DiskMan + Unishare |
| 容器 | Docker + docker-compose + LuCI dockerman |
| 限速 | EQoS（按设备上下行限速） |
| 其他 | mwan3 多拨、DDNS、UPnP、ttyd、argon 主题（中文）、N1 WiFi（brcmfmac SDIO） |

> 私人配置（lucky 数据、easytier config.toml、PPPoE 账号、MAC 绑定）**不在固件内预置**，首次开机后在 LuCI 里自行配置。

## 产物说明（Actions Artifact）

| 文件 | 用途 |
| --- | --- |
| `openwrt_s905d_n1_*.img.gz` | **N1 直刷镜像（推荐）**：ophub 重打包，含 s905d u-boot + 同版本预编译 Amlogic 内核（ARCH_MESON / DEBUG_INFO_BTF / MMC_MESON_GX 已核实） |
| `immortalwrt-*-armsr-armv8-generic-squashfs-combined.img.gz` | armsr 原始镜像（UEFI 通用，N1 不能直接引导，仅作 rootfs 备胎） |

## 刷机说明（N1）

前提：N1 已刷过第三方 u-boot（Armbian/OpenWrt 折腾过的盒子基本都满足）。

1. 把 `openwrt_s905d_n1_*.img.gz` 解压成 `.img` 写入 U 盘：
   `dd if=openwrt_s905d_n1_*.img of=/dev/你的U盘 bs=4M conv=fsync`（或用 balenaEtcher）
2. N1 插 U 盘、牙签捅复位键上电，从 U 盘启动
3. 进系统后可从 U 盘写 eMMC：`ophub` 相关工具（`dd` 镜像到 `/dev/mmcblk2`）或按社区惯例一键安装
4. LAN 口接电脑，访问 `http://192.168.99.1`（LuCI 账号 `root`，密码首次登录设置）

## 编译集成要点

- **N1 引导链**：armsr 内核无 Meson 平台驱动 → CI 末尾用 [ophub/amlogic-s9xxx-openwrt](https://github.com/ophub/amlogic-s9xxx-openwrt) `remake -b s905d` 把 rootfs 与**同版本**预编译 Amlogic 内核重打包（内核版本严格匹配，kmod 才能 load）
- **daed eBPF**：ophub 6.12 stable 内核 `config-6.12` 已核实 `DEBUG_INFO_BTF=y`；armsr 侧同步开启 BTF 并钉 `DAED_USE_KERNEL_BTF`
- **iStore**：jjm2473 packages/luci fork（iStoreOS 25.12 官方同款）+ 显式 `xz-utils=y`（防 tar 依赖死锁连坐）
- **xray-core**：钉 26.7.28（26.9.9 需要 go1.27，25.12 只有 go1.26.8）
- **瘦身**：第三方 feed 按需安装，mihomo-alpha/clashoo/momo 黑名单双重校验

## 免责声明

仅供个人学习与家用折腾使用，请遵守当地法律法规。
