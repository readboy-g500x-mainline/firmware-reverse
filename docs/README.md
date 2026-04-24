
读书郎 G500X 主线 Linux 移植 — 固件逆向文档

1. 设备概述

项目 详情
设备型号 读书郎 G500X (Readboy G500X)
SoC Qualcomm MSM8937 (八核 Cortex-A53, 4×1.4GHz + 4×1.1GHz)
GPU Adreno 505
PMIC PM8937 + PMI8937
RAM 3 GB LPDDR3 (原厂系统 MemTotal: 2944832 kB)
存储 64 GB eMMC (FORESEE NCEMASLD-64G)
显示屏 1200×1920 LCD (MIPI DSI, 面板型号 XZ_NT51021)
触摸屏 I2C 触摸 (Goodix GT9xx / FocalTech / Synaptics)

2. 硬件信息

· 内存颗粒: FORESEE NCBMS-0104 LPDDR3
· 物理内存布局: 起始地址 0x40000000, 长度 0xc0000000 (3 GB)
· eMMC: mmc0: 内置 eMMC (sdhc_1), mmc1: 外部 SD 卡 (sdhc_2)
· 调试串口: UART2 (GPIO4 TX, GPIO5 RX), 位于 0x78b0000

3. 已提取的原厂固件

3.1 设备树 (Device Tree)

· 文件: devicetree/original/02_dtbdump_Qualcomm_Technologies,_Inc._MSM8937-PMI8937_QRD_SKU2.dts
· 来源: 从原厂 boot.bin 提取并反编译
· 用途: 作为主线设备树的参考，包含完整的硬件描述（引脚、电源、显示、触摸等）

3.2 开机第一屏 (Splash)

· 文件: splash.bin
· 格式: 高通 SPLASH!! 格式，1200×1920，原始像素数据
· 状态: 已提取，已尝试解包（RGBA/BGRA/stride 对齐仍在调试中）

3.3 诊断日志

· 文件: logdump.bin (64 MB)
· 描述: 系统崩溃时保存的诊断日志，包含内核日志、寄存器状态、协处理器崩溃转储
· 状态: 已提取，已用 strings 分析（包含约 22920 行可读文本），尚未深入排查

3.4 其他分区

· lk2nd: 已刷入 lk2nd 引导加载程序，用于启动主线内核
· boot.bin: 原厂 boot 分区镜像
· recovery.bin: 原厂 recovery 分区镜像

4. 主线内核移植进展

4.1 内核源码

· 当前使用: linux-msm89x7-6.19.3 (已编译)
· 社区推荐: msm89x7-mainline/linux 
· 编译工具链: aarch64-linux-gnu- (GCC 15.1.0)

4.2 设备树

· 主线设备树: msm8937-readboy-g500x.dts
· 基于: 主线 msm8937.dtsi + pm8937.dtsi + pmi8950.dtsi
· 状态:
  · 内存配置正确 (3 GB, 0x40000000 起始)
  · 保留内存区域已精简，不再出现重叠警告
  · 调试串口 (UART2) 已启用
  · MSM DRM 驱动已通过 modprobe.blacklist=msm 禁用
  · 其他外设 (MMC, USB, WCNSS) 已临时禁用

4.3 启动状态

· 启动流程: lk2nd 正常工作, 可成功加载主线内核和设备树
· 当前卡点: 内核启动到 ASID allocator initialised with 65536 entries 后发生 synchronous external abort 崩溃
· 原因分析: 当前内核版本 6.19.3-msm89x7 缺少 MSM8937 关键时钟驱动支持，特别是 CONFIG_QCOM_RPMCC (RPM 时钟控制器) 和 CONFIG_QCOM_GCC_MSM8937 (全局时钟控制器)

4.4 下一步计划

1. 切换到社区维护的内核分支: msm89x7-mainline/linux (分支 barni2000/6.14-sdm439)
2. 配置并编译新内核: 启用关键时钟驱动 (CONFIG_QCOM_RPMCC, CONFIG_QCOM_GCC_MSM8937)
3. 测试启动: 使用精简设备树启动内核，预期成功进入命令行
4. 逐步添加外设: MMC → USB → Display → Touchscreen → Audio → WLAN/BT

5. 已收集的资源

资源 用途
原厂 dts 硬件参考
原厂 boot.bin 分区布局、启动参数
splash.bin 开机画面逆向
logdump.bin 崩溃诊断
lk2nd 引导加载程序

6. 启动参数 (Bootargs)

原厂启动参数:

```
sched_enable_hmp=1 console=ttyHSL0,115200,n8 androidboot.console=ttyHSL0
androidboot.hardware=qcom msm_rtb.filter=0x237 ehci-hcd.park=3
androidboot.bootdevice=7824900.sdhci earlycon=msm_hsl_uart,0x78B0000
androidboot.emmc=true androidboot.serialno=7d16aa398760
androidboot.baseband=msm
```

当前主线启动参数:

```
console=ttyMSM0,115200n8 earlycon ignore_loglevel loglevel=8
clk_ignore_unused modprobe.blacklist=msm
```

7. 联系方式

· GitHub: readboy-g500x-mainline
· 维护者: S-lkno


