---
title: Docker：部署数据科学栈Jupyter
tags: []
id: '2038'
categories:
  - - uncategorized
date: 2022-10-02 07:45:21
---

## 持久化镜像存储

```yml
version: '3.3'
services:
    datascience-notebook:
        ports:
            - '57002:8888'
        container_name: jupyterR
        restart: always
        image: 'jupyter/datascience-notebook:r-4.1.3'
        command: start-notebook.sh --NotebookApp.token='***'
```

*   mkdir -p ~/datascience && cd ~/datascience
*   nano docker-compose.yml
*   sudo docker-compose up -d
*   sudo docker-compose logs
*   sudo docker cp -a jupyterR:/opt /home/limour/upload/opt
*   sudo docker cp -a jupyterR:/home/jovyan /home/limour/upload/home
*   sudo docker-compose down && sudo docker volume prune

## 启动镜像

```R
version: '3.3'
services:
    datascience-notebook:
        ports:
            - '57002:8888'
        container_name: jupyterR
        restart: always
        volumes:
            - '/home/limour/upload:/home/jovyan/upload'
            - '/home/limour/upload/opt/opt:/opt'
            - '/home/limour/upload/home/jovyan:/home/jovyan'
        image: 'jupyter/datascience-notebook:r-4.1.3'
        command: start-notebook.sh --NotebookApp.token='***'
```

*   nano docker-compose.yml
*   sudo chmod 777 -R /home/limour/upload/
*   sudo docker-compose up -d
*   sudo docker-compose logs
*   NPS内网穿透加NPM面板反代

## R包镜像

```R
options(BioC_mirror="https://mirrors.ustc.edu.cn/bioc/") ##指定镜像，这个是中国科技大学镜像
options("repos" = c(CRAN="https://mirrors.tuna.tsinghua.edu.cn/CRAN/")) ##指定install.packages安装镜像，这个是清华镜像
options(ggrepel.max.overlaps = Inf)
```

*   nano .Rprofile
*   options()$repos ## 查看使用install.packages安装时的默认镜像
*   options()$BioC\_mirror ##查看使用bioconductor的默认镜像

## conda镜像

[Anaconda 镜像使用帮助](https://mirror.tuna.tsinghua.edu.cn/help/anaconda/)；[Docker安装A面板](https://occdn.limour.top/2285.html)

*   nano .condarc
*   conda clean -i
*   conda create -n seurat -c conda-forge r-seurat=4.1.1 -y
*   conda activate seurat
*   conda install -c conda-forge r-tidyverse -y
*   conda install -c conda-forge r-irkernel -y
*   Rscript -e "IRkernel::installspec(name='seurat', displayname='r-seurat')"
*   wget -e "https\_proxy=http://172.17.0.1:20171" https://github.com/chris-mcginnis-ucsf/DoubletFinder/archive/refs/heads/master.zip -O DoubletFinder-master.zip
*   上面的"https\_proxy=http://172.17.0.1:20171"来自容器A面板