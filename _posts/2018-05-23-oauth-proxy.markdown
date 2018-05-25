---
layout: post
title:  "OAuth Proxy"
author: 윤상준
date: 2018-05-23 02:01:05 +0900
categories: authentication
---

OpenID Connect Provider와 연계하여 애플리케이션의 인증을 해야하는 경우가 있습니다. 하지만 애플리케이션에서 OpenID Connect Provider와 연계하기 위한 플러그인을 제공하지 않거나 소스 코드를 수정할 수 없는 경우가 있습니다.(예 : Kibana). 이 경우 Keycloak Proxy와 같은 OAuth Proxy를 사용하여 이런 애플리케이션의 인증/인가를 처리할 수 있습니다.

이 문서에서는 Keycloak Proxy를 활용하여 애플리케이션의 인증을 처리하는 방법에 대한 가이드를 제공합니다.
아래의 "데모 애플리케이션"과 "Keycloak Proxy" 배포 따라하기를 통해 Keycloak Proxy를 활용한 인증방법을 쉽게 이해할 수 있습니다.

## 데모 애플리케이션 배포하기

### 구조


### 다운로드

```
$ git clone https://github.com/YunSangJun/keycloak-proxy-demo
$ cd keycloak-proxy-demo
```

### 배포

```
$ kubectl apply -f deployment.yaml
$ kubectl apply -f service.yaml
```

### 배포 확인

```
$ kubectl get po,svc
NAME                       READY     STATUS    RESTARTS   AGE
po/demo-85cdbcc8c7-6pkbv   1/1       Running   0          10s

NAME               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
svc/demo-service   ClusterIP   172.21.189.11   <none>        80/TCP    10s
```

### 접속 확인

`http://127.0.0.1:8080/user`와 `http://127.0.0.1:8080/admin`엡 접속해봅니다.
"Hello User!"와 "Hello Admin!" 메세지를 볼 수 있습니다.
현재는 `/user`와 `/admin` endpoint에 별도의 인증없이 접속 할 수 있습니다.

다음 과정에서는 Keycloak(OpenID Connect Provider)을 통해 인증/인가를 거친 사용자만 `/admin` endpoint에 접속할 수 있도록 설정해 보겠습니다.

```
$ kubectl port-forward demo-85cdbcc8c7-6pkbv 8080:8080
Forwarding from 127.0.0.1:8080 -> 8080
```

## Keycloak Proxy 배포하기
