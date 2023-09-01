---
title: 【树莓派】将U盘格式化为F2FS格式并挂载
tags: []
id: '2107'
categories:
  - - uncategorized
date: 2023-01-12 11:15:20
---

F2FS是专为具有NAND闪存设备的硬件设计的，比如NVMe SSD，SmartMediaCard（SD卡）等，在使用这类设备情况下具有更快的读写速度。

*   sudo ls /dev/sd\* #找到刚插入的U盘设备，比如/dev/sda
*   sudo mkfs.f2fs -f -O inode\_checksum -O extra\_attr -O verity -O lost\_found /dev/sda
*   sudo nano /etc/rc.local # 添加下面的命令
*   sudo /bin/bash -c "echo '- - -' > /sys/class/scsi\_host/host0/scan"
*   sudo /bin/mount -t f2fs /dev/sda /home/share/