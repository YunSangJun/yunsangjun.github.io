---
layout: post
title:  "Dockerfile 작성 best practices"
author: sj
date: 2020-02-26
categories: docker
tags:
- container
- docker
---

## Overview

Dockerfile 작성 시 아무렇게나 작성하다보면 [이미지 레이어](/docker/2018/05/26/docker-image-layer.html)가 불필요하게 늘어나고 이미지 크기도 증가합니다. 이런 식으로 Dockerfile 작성 시 몇가지 피해야 할 케이스들이 있습니다.

Docker 공식 문서를 보면 [Best practices for writing Dockerfiles](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/) 라는 글이 있는데 Dockerfile 작성 시 주의 사항과 모범적인 예제를
가이드합니다.

저도 Dockerfile을 주기적으로 작성하지 않다보니 자꾸 잊어버리고 실수를 반복하는 경우가 많아서 
위에서 언급한 공식 문서의 일부분을 정리해 놓고 작성 시 리뷰를 하고 있습니다.

## Multi-stage 빌드 활용

Dockerfile을 작성할 때 Multi-stage 빌드를 활용하는 것이 좋습니다.
Multi-stage 빌드를 사용하게 되면 이미지 크기를 효율적으로 줄일 수 있기 때문입니다.

<p class="warning-title">경고</p>
<p class="warning-content">
Multi-stage 빌드 기능을 사용하기 위해 "Docker 17.05" 이상 버전이 필요합니다.<br/><br/>

Openshift 3.x 버전은 Docker 1.13.1 버전을 사용하지만 "imagebuilder"를 통해 이 기능을 지원한다고 합니다.<br/>
https://github.com/openshift/origin/issues/21627<br/><br/>

Docker 버전 체계는 "~1.13.1" 버전으로 이후로 "17.x ~" 로 릴리스되고 있습니다.<br/>
- "~1.13.1" : https://docs.docker.com/release-notes/docker-engine/<br/>
- "17.x ~" : https://docs.docker.com/engine/release-notes/<br/>
</p>

### TL;DR;
좀 더 자세한 내용을 설명하자면 내용이 길어지기 때문에 먼저 예제를 살펴보겠습니다.
자세한 내용은 뒤에 이어지는 "Multi-stage 빌드 사용 배경"을 읽어보시면 됩니다.

아래 multi-stage 빌드를 활용한 Dockerfile 예제가 있습니다.

- Go build를 위한 "golang:latest" 이미지는 최종 이미지에 포함되지 않습니다. 
- 최종 이미지인 "alpine:latest" 에서는 빌드 결과물인 실행파일(app)만 복사해옵니다.
- 최종 이미지에서는 불필요한 Go 패키지 및 소스 코드를 가지고 있지 않습니다.
- [예제 코드](https://github.com/YunSangJun/best-practices-dockerfile/tree/master/multi-stage-build)

```Dockerfile
# 빌드 이미지 선언(최종 이미지에는 포함되지 않음)
FROM golang:latest AS builder
 
# Working directory 지정
WORKDIR /go/src/
 
# 소스 코드 복사
COPY src/app.go .
 
# 소스 코드 빌드
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o app .
 
# Base 이미지 선언(최종 이미지)
FROM alpine:latest
 
# 인증서 추가
RUN apk --no-cache add ca-certificates
 
# Working directory 지정
WORKDIR /root/
 
# Build 결과 복사(빌드 이미지로 부터)
COPY --from=builder /go/src/app .
 
# 바이너리 실행
CMD ["./app"]
```

### Multi-stage 빌드 사용 배경
일반적으로 Dockerfile은 소스코드를 빌드하고 결과물을 실행하는 방식으로 작성합니다.

단순하게 Dockerfile을 작성한다면 실행 환경으로 사용할 리눅스 이미지(예: ubuntu)에
빌드에 필요한 패키지, 라이브러리를 다운로드 받아서 이 이미지를 하나의 이미지로 관리할 것입니다.
예를 들어 go 언어로 작성된 소스코드를 빌드하고 ubutun 환경에서 실행한다면 이미지는 "golang:ubuntu"와 같이 됩니다.

하지만 앱을 실행할 때는 빌드 단계의 레이어는 사용하지 않지만 용량만 차지하는 불필요한 자원입니다.
그래서 빌드, 실행 단계를 분리할 수 있는 빌드 패턴이 나왔습니다.
빌드, 실행 단계의 Dockerfile을 각각 작성하고 이를 한번에 실행할 수 있는 스크립트를 만드는 방식인데
자세한 내용은 [Before multi-stage builds](https://docs.docker.com/develop/develop-images/multistage-build/#before-multi-stage-builds) 라는 글을 참고하세요.

하지만 이마저도 Dockerfile을 두개로 관리해야하고 스크립트도 작성해야되다 보니 번거로운 작업이라 느껴졌고
그래서 "Alex Ellis" 라는 엔지니어가 이를 해결할 수 있는 "multi-stage build" 기능을 PR로 올려 Merge가 됐습니다.
이와 관련된 [Builder pattern vs. Multi-stage builds in Docker](https://blog.alexellis.io/mutli-stage-docker-builds/) 블로그가 있으니 더 궁금하시면 참고하세요.

여튼 그래서 어쨋거나.. 
우리는 multi-stage 빌드 기능을 사용해서 이미지의 크기를 효율적으로 줄일 수 있다라는 점만 기억하면 됩니다.

## 이미지 레이어 최소화

Docker 1.10 이상 버전에서 "RUN", "COPY", "ADD" 명령어는 레이어를 생성합니다.(=이미지 사이즈 증가)

따라서 명령어는 가능한 아래와 같이 한줄로 작성하세요.

명령을 이어서 실행할때는 ";" 보다는 "&&" 을 사용하세요.

```Dockerfile
# Do not
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get install -y nginx
 
COPY a.txt /tmp
COPY b.txt /tmp
 
# Do
RUN apt-get update && apt-get install -y \
    curl \
    nginx \
    && apt-get clean all
 
COPY a.txt b.txt /tmp
```

## Multi-line argument 정렬

Argument를 여러 줄로 작성하는 경우 알파벳 순으로 정렬하세요.

```Dockerfile
# Do not
RUN apt-get update && apt-get install -y curl wget git net-tools dnsutils telnet vim \
    && apt-get clean all
 
# Do
RUN apt-get update && apt-get install -y \
    curl \
    dnsutils \
    git \
    net-tools \
    telnet \
    wget \
    && apt-get clean all
```

## Hard code 제거

변경 될 수 있는 속성은 ENV(Environment Variable)로 정의해서 사용하세요.

ENV로 정의한 변수는 컨테이너 실행 시 주입할 수 있습니다.

```Dockerfile
# Do not
RUN apt-get update && apt-get install -y curl \
&& curl -OL https://github.com/example/example.tar.gz --proxy http://example-proxy.com
 
# Do
ENV http_proxy http://example-proxy.com
RUN apt-get update && apt-get install -y curl \
&& curl -OL https://github.com/example/example.tar.gz --proxy $http_proxy
 
$ docker run --env http_proxy="http://another-proxy.com"
```

## 파일 복사

파일을 다운로드 한 후에 다음 layer에서 삭제하면 중간 layer에는 파일이 그대로 남게됩니다.

따라서 다운로드 받아서 실행하고 삭제하는 것까지 한줄로 실행하세요.

```Dockerfile
# Do not
COPY openshift-origin-client-tools-v3.11.0-0cbc58b-linux-64bit.tar.gz .
RUN tar zxvf openshift-origin-client-tools-v3.11.0-0cbc58b-linux-64bit.tar.gz
RUN rm -rf openshift-origin-client-tools-v3.11.0-0cbc58b-linux-64bit.tar.gz
 
# Do not
ADD https://github.com/openshift/origin/releases/download/v3.11.0/openshift-origin-client-tools-v3.11.0-0cbc58b-linux-64bit.tar.gz .
RUN rm -rf openshift-origin-client-tools-v3.11.0-0cbc58b-linux-64bit.tar.gz
  
# Do
RUN curl -OL https://github.com/openshift/origin/releases/download/v3.11.0/openshift-origin-client-tools-v3.11.0-0cbc58b-linux-64bit.tar.gz \
    && tar zxvf openshift-origin-client-tools-v3.11.0-0cbc58b-linux-64bit.tar.gz \
    && rm -rf openshift-origin-client-tools-v3.11.0-0cbc58b-linux-64bit.tar.gz
```

## ADD or COPY

ADD와 COPY 명령은 파일을 복사하는 기능을 공통적으로 가지고 있습니다.

하지만 파일을 복사하기 위한 용도라면 COPY 명령을 사용하세요.

```Dockerfile
# Do not
ADD nginx.conf /etc/nginx/nginx.conf
  
# Do
COPY nginx.conf /etc/nginx/nginx.conf
```

<p class="tip-title">참고</p>
<p class="tip-content">
ADD 명령어로 압축(tar) 파일을 복사하는 경우 추출(extraction)도 함께 수행됩니다.<br/>

따라서 단순히 복사만을 하는 경우는 COPY 명령을 사용하는 것이 좋습니다.
</p>