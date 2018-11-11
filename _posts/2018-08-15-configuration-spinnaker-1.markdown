---
layout: post
title:  "Spinnaker 설정 #1 도메인 주소를 통한 대시보드 접속"
author: 윤상준
date: 2018-08-15
categories: spinnaker
tags:
- spinnaker
- kubernetes
- cicd
- devops
---

## 도메인 주소를 통한 대시보드 접속하기

로컬 환경이 아닌 외부에서 도메인 주소를 통해 Spinnaker에 접속하기 위해서 추가 설정이 필요합니다.

이 페이지에서는 Kubernetes provider를 기준으로 설명하겠습니다.
Kubernetes 환경에서는 외부에 서비스를 노출시키기 위해 Ingress를 사용합니다.

먼저 Ingress에서 사용할 도메인을 Haryard에 설정합니다.

```
$ hal config security ui edit --override-base-url http://spinnaker.example.com
$ hal config security api edit --override-base-url http://spinnaker-api.example.com
```

Spinnaker를 다시 배포합니다.

```
$ hal deploy apply
```

다음으로 `ingress.yaml`을 작성합니다.

```
$ vi ingress.yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: spinnaker
spec:
  rules:
  - host: spinnaker.example.com
    http:
      paths:
      - backend:
          serviceName: spin-deck
          servicePort: 9000
  - host: spinnaker-api.example.com
    http:
      paths:
      - backend:
          serviceName: spin-gate
          servicePort: 8084
```

Spinnaker가 설치된 Kubernetes cluster에 Ingress를 배포합니다.

```
$ kubectl apply -f ingress.yaml
```

이제 `spinnaker.example.com`에 접속해 Spinnaker 대시보드를 사용할 수 있습니다.

![](/blog/assets/images/spinnaker/spinnaker-dashboard.png)
