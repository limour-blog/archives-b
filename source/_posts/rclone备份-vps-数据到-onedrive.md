---
title: Rclone备份 VPS 数据到 onedrive
tags: []
id: '2117'
categories:
  - - uncategorized
date: 2023-01-30 07:28:41
---

运营5年的HostDare跑路了，还好上面没有重要数据。如今大厂VPS昂贵根本买不起，oneman的VPS随时有跑路风险，只能靠每天备份来勉强度日了。

## [Rclone挂载onedrive](https://occdn.limour.top/2083.html#Rclone)

*   [Rclone挂载onedrive](https://occdn.limour.top/2083.html#Rclone)
*   rclone lsd od\_lk:backup/lk #预先创建了备份存放的目录，这里查看一下目录

## 备份脚本

```bash
#!/bin/bash
tar -zcPf /root/tmp/ngpm_live.tar.gz /root/base/NGPM/letsencrypt/live
# tar -tzvPf /root/tmp/ngpm_live.tar.gz
rclone sync --progress --ignore-errors --transfers=2 \
--exclude='/.*/**' \
--exclude='/.*' \
--exclude='/app/ServerStatus/serverstatus-monthtraffic/**' \
--exclude='/app/WordPress/www/wp-content/cache/**' \
--exclude='/base/NGPM/letsencrypt/live/**' \
--exclude='/base/NGPM/data/logs/**' \
/root od_lk:backup/lk 
```

*   nano /root/backup.sh && chmod +x /root/backup.sh
*   /root/backup.sh
*   crontab -e
*   30 2 \* \* 2,4,6 /root/backup.sh
*   crontab -l