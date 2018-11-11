---
layout: post
title:  "Ingress TLS Secret 생성하기"
author: 윤상준
date: 2018-05-27
categories: kubernetes
tags:
- kubernetes
- container
- ingress
- https
- tls
- secret
---

이 페이지에서는 Ingress에 TLS를 적용하기 위한 Secret을 생성하는 방법을 가이드합니다.

## TLS Secret 생성

아래 명령을 실행하여 TLS Secret을 생성합니다.

```
$ kubectl create secret tls example-tls --cert example.crt --key example.key
```

또는 아래와 같이 yaml 파일 형태로 생성할 수 있습니다.

```
$ vi secret.yaml
apiVersion: v1
data:
  tls.crt: base64 encoded cert
  tls.key: base64 encoded key
kind: Secret
metadata:
  name: example-tls
type: Opaque

$ kubectl create -f secret.yaml
```

## TLS Secret을 Ingress에 적용

아래와 같이 Ingress의 secretName 속성에 example-tls을 적용합니다.
이제 Ingress를 배포한 후 host로 접속하면 TLS가 적용된 것을 확인할 수 있습니다.

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: example-ingress
spec:
  rules:
  - host: example.com
    http:
      paths:
      - backend:
          serviceName: example
          servicePort: 80
  tls:
  - hosts:
    - example.com
    secretName: example-tls
```
