---
title: 初次尝试docker build
tags: []
id: '2149'
categories:
  - - uncategorized
date: 2023-02-28 15:43:19
---

```dockerfile
# set alpine as the base image of the Dockerfile
FROM alpine:latest

# update the package repository and install Tor
RUN apk update && apk --no-cache add tor

# Copy over the torrc created above and set the owner to `tor`
COPY torrc /etc/tor/torrc
RUN chown -R tor /etc/tor

# Set `tor` as the default user during the container runtime
USER tor

# Set `tor` as the entrypoint for the image
ENTRYPOINT ["tor"]

```

```yml
version: '3.3'
services:
    tor:
        container_name: tor
        restart: always
        image: limour/tor
    
networks:
  default:
    external: true
    name: sswitch
```

*   docker login -u username -p 'password'

*   mkdir -p ~/app/tor && cd ~/app/tor && nano Dockerfile && nano docker-compose.yml

*   echo 'SocksPort 0.0.0.0:9050' > torrc

*   docker build -t limour/tor .

*   docker image ls grep limour/tor

*   sudo docker network create sswitch

*   sudo docker-compose up -d && sudo docker-compose logs

*   \# sudo docker-compose down

*   sudo docker-compose logs tail

### 测试效果

*   docker run --rm --net=sswitch alpine/curl https://check.torproject.org/api/ip

*   docker run --rm --net=sswitch alpine/curl --socks5 tor:9050 https://check.torproject.org/api/ip