---
title: Docker安装RStudio
tags:
  - Docker
  - raspberrypi
  - Rstudio
id: '575'
categories:
  - - Raspberry Pi
date: 2021-03-10 20:33:17
---

```
proxychains4 docker pull arturklauser/raspberrypi-rstudio-server:1.2.5019-buster-buildx
docker run --rm --name rserver -v /mnt/data:/home/rstudio -p 18787:8787 -d arturklauser/raspberrypi-rstudio-server:1.2.5019-buster-buildx
```

```
[web07]
type = http
local_ip = 127.0.0.1
local_port = 18787
use_compression = true
subdomain = rstudio4
```

`user name: rstudio  
passwd: raspberry`

[**  
raspberrypi-rstudio**：https://github.com/ArturKlauser/raspberrypi-rstudio](https://github.com/ArturKlauser/raspberrypi-rstudio)