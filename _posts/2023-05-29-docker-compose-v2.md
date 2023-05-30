---
layout: post
title: Install docker-compose V2
featured-img: emile-perron-190221
category: [Software Engineering]
summary: How to install docker-compose V2.
---

# Description
- Install docker-compose version 2

# Prerequisites
- Linux based OS with docker CLI

# Install Steps 
(1) Set environment variable
```
DOCKER_CONFIG=${DOCKER_CONFIG:-$HOME/.docker}
DOCKER_COMPOSE_VERSION=2.18.1
```
The latest version of docker-compose now is 2.18.1

(2) Create a folder
```
mkdir -p $DOCKER_CONFIG/cli-plugins
```

(3) Download docker-compose file
```
curl -SL "https://github.com/docker/compose/releases/download/v$(echo $DOCKER_COMPOSE_VERSION)/docker-compose-$(uname -s)-$(uname -m)" -o $DOCKER_CONFIG/cli-plugins/docker-compose
```

(4) Change the permission
```
chmod +x $DOCKER_CONFIG/cli-plugins/docker-compose
```
(5) Check installed version
```
docker compose version
```