---
layout: post
title:  "Docker 이미지 레이어란?"
author: sj
date: 2018-05-26
categories: docker
tags:
- container
- docker
- image
- layer
---

Docker Image를 pull, push 하다보면 layer라는 용어가 나옵니다. Docker 에서 사용하는 layer 구조가 무엇이고 왜 사용할까요?

```
$ docker pull nginx
Using default tag: latest
latest: Pulling from library/nginx
2a72cbf407d6: Pulling fs layer
04b2d3302d48: Pulling fs layer
e7f619103861: Pulling fs layer
...
2a72cbf407d6: Pull complete
04b2d3302d48: Pull complete
e7f619103861: Pull complete
Digest: sha256:18156dcd747677b03968621b2729d46021ce83a5bc15118e5bcced925fb4ebb9
Status: Downloaded newer image for nginx:latest
```

## Docker Image Layer 구조란?

Dockerfile을 build 하면 정의된 명령어에 따라 Docker 이미지가 생성됩니다. 하나의 이미지로 보이지만 내부적으로는 여러개의 이미지가 층층히 쌓여있는 layer 구조입니다. 좀 더 쉽게 예를 들어 보겠습니다.

아래 예제는 nginx image를 build하기 위한 Dockerfile입니다. step1~6까지 총 6개의 명령으로 이루어져 있습니다.

```
<Dockerfile>
# step1. Base image
FROM ubuntu:latest

# step2. Install Nginx.
RUN \
  apt-get update && \
  apt-get install -y nginx

# step3. Define mountable directories.
VOLUME ["/etc/nginx/sites-enabled", "/etc/nginx/certs", "/etc/nginx/conf.d", "/var/log/nginx", "/var/www/html"]

# step4. Define working directory.
WORKDIR /etc/nginx

# step5. Define default command.
CMD ["nginx"]

# step6. Expose port.
EXPOSE 80
```

이 Dockerfile을 build 해보겠습니다. 아래와 같이 각 명령어 단계마다 이미지가 생성됩니다.

```
$ docker build -t nginx .
Sending build context to Docker daemon  2.048kB
Step 1/6 : FROM ubuntu:latest
...
 ---> c9d990395902    # 1번 이미지
Step 2/6 : RUN   apt-get update &&   apt-get install -y nginx
...
 ---> 90100bc32c07    # 2번 이미지
Step 3/6 : VOLUME ["/etc/nginx/sites-enabled", "/etc/nginx/certs", "/etc/nginx/conf.d", "/var/log/nginx", "/var/www/html"]
...
 ---> 8ea7a4443b1e    # 3번 이미지
Step 4/6 : WORKDIR /etc/nginx
...
 ---> 37773795f83d    # 4번 이미지
Step 5/6 : CMD ["nginx"]
...
 ---> 4f8728cb93dc    # 5번 이미지
Step 6/6 : EXPOSE 80
...
 ---> bec972ccfa66    # 6번 이미지
Successfully built b5d127ff2140
Successfully tagged nginx:latest
```

## 왜 사용할까요?

그렇다면 이런 Image layer 구조는 왜 사용할까?
Nginx 이미지를 기반으로 Web App을 만들었다고 가정해보자. App source를 수정할 때 마다 전체 이미지를 다시 다운로드 받는다면 매우 비효율적이라 생각된다. 하지만 Docker Image는 layer 구조로 되어있기 때문에 Base image인 nginx image layer는 다운로드 받지 않고 변경된 source layer만 받게 된다. 이런 이유로 Docker Image는 layer 구조로 설계되어 있다.
