---
title: Docker安装vscode-web
tags: []
id: '2163'
categories:
  - - uncategorized
date: 2023-03-13 10:53:15
---

## 安装vscode-web

```yml
version: "2.1"
services:
  code-server:
    image: linuxserver/code-server:latest
    container_name: code-server
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Shanghai
      - PASSWORD=password
      - SUDO_PASSWORD=password
      - PROXY_DOMAIN=code-server.my.domain #optional
      - DEFAULT_WORKSPACE=/config/workspace #optional
    volumes:
      - ./config:/config
    ports:
      - 2441:8443
    restart: unless-stopped
```

*   mkdir -p ~/app/vscode && cd ~/app/vscode && nano docker-compose.yml

*   sudo docker-compose up -d && sudo docker-compose logs

## vscode-web配置代理和中文

*   打开Visual Studio Code，点击Manage，在列表中选择Settings

*   在弹出的搜索框中输入"proxy"，即可看到代理的配置项"Http:Proxy"

*   宿主机获取docker0的ip: ip address grep docker0

*   然后docker内设置代理 http://docker0的ip:port

*   拓展内搜索zh-cn，安装中文界面拓展

## vscode-web安装conda

*   回到WORKSPACE，ctrl+~ 调出终端

*   sudo sed -i 's/archive.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list

*   sudo apt update

*   sudo apt install wget

*   [安装conda](https://occdn.limour.top/2278.html#%E5%85%88%E7%BB%99%E8%99%9A%E6%8B%9F%E6%9C%BA%E8%A3%85%E4%B8%AAconda)

*   wget https://mirrors.bfsu.edu.cn/anaconda/miniconda/Miniconda3-py39\_23.1.0-1-Linux-x86\_64.sh -O conda\_install.sh

*   chmod +x conda\_install.sh && ./conda\_install.sh

*   宿主机执行 sudo docker-compose restart # 重启容器使conda生效

*   nano -K ~/.condarc # [清华镜像源](https://mirrors.tuna.tsinghua.edu.cn/help/anaconda/)

*   conda clean -i # 清除索引缓存，保证用的是镜像站提供的索引

## conda安装nodejs

*   conda create -n node -c conda-forge nodejs

*   conda activate node

*   npm config set registry https://registry.npmmirror.com

## vscode-web使用git

*   npm create astro@latest

*   git config --global user.email "youremail"

*   git config --global user.name "yourname"

*   git branch -M main && git add . && git commit -m 'Initial commit'

*   git remote add origin https://github.com/Limour-dev/chatGPT.git

*   git push --set-upstream origin main --force # [Creating a personal access token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token)

*   git config --global credential.helper cache

*   git push

## hello world

```js
---
const search = Astro.url.searchParams.get('search')!  '';
---
<h1>{search}</h1>
```

*   [Enabling SSR in Your Project](https://docs.astro.build/en/guides/server-side-rendering/#enabling-ssr-in-your-project)

*   编辑 chatGPT/src/pages/index.astro

*   npm run dev

*   https://vscode.domain/proxy/3000/?search=hello%20world