---
title: qemu模拟器
date: 2019-12-12
tags:
---

## 启动

### Legacy

常见bootloader

grub引导流程:

1. BIOS将控制权交给硬盘的主引导区，即MBR。
2. MBR中的bootloader(stage1)通过内置的地址加载stage1_5
3. bootloader通过stage1_5的内容，将分区中的stage2加载
4. stage2此时就可以在文件系统中将grub.conf文件加载，让用户看到选项界面。

引导基本知识

grub安装在磁盘

grub安装在分区

启动到grub菜单，按c进入命令行界面，这里可以手动配置

### UEFI

设备 -> esp