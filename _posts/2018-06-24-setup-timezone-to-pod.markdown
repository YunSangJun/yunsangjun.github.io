---
layout: post
title:  "Pod에 타임존 설정하기"
author: 윤상준
date: 2018-06-24
categories: kubernetes
tags:
- kubernetes
- container
- timezone
---

이 페이지는 Pod에 타임존을 설정하는 방법에 대해 설명합니다.

클라우드 환경에서는 기본 타임존이 UTC로 설정되어 있는 경우가 많습니다.

Kubernetes 클러스터의 경우에도 호스트의 타임존이 UTC로 설정되어 있으면 Pod의 타임존 또한 UTC로 설정됩니다.

아래의 가이드를 참고하여 특정 Pod의 타임존을 변경할 수 있습니다.

## 준비하기

먼저 테스트를 위한 샘플앱을 Kubernetes 클러스터에 배포합니다.

```
$ kubectl run nginx --image=nginx --port=80
```

## 현재 타임존 확인

아래 명령을 실행하여 현재 Pod의 타임존을 확인합니다.

현재 타임존이 UTC로 설정되어 있는 것을 확인할 수 있습니다.

```
$ kubectl get po
NAME                     READY     STATUS    RESTARTS   AGE
nginx-7587c6fdb6-wl962   1/1       Running   0          11s

$ kubectl exec nginx-7587c6fdb6-wl962 -it -- date
Sun Jun 24 07:30:17 UTC 2018
```

Pod의 로그 또한 UTC 시간을 기준으로 조회되는 것을 확인할 수 있습니다.

```
$ kubectl logs -f nginx-7587c6fdb6-wl962
127.0.0.1 - - [24/Jun/2018:07:30:34 +0000] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.181 Safari/537.36" "-"
```

## 타임존 변경

아래와 같이 타임존을 변경하기 위해 deployment 리소스를 편집합니다.

```
$ kubectl get deploy
NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx     1         1         1            1           2m

$ kubectl edit deploy nginx
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx
spec:
  ...
  template:
    ...
    spec:
      containers:
      ...
        volumeMounts:
        - mountPath: /etc/localtime
          name: timezone-config
      volumes:
      - hostPath:
          path: /usr/share/zoneinfo/Asia/Seoul
        name: timezone-config
```

## 변경된 타임존 확인

아래 명령을 실행하여 변경된 Pod의 타임존을 확인합니다.

타임존이 KST로 변경된 것을 확인할 수 있습니다.

```
$ kubectl get po
NAME                    READY     STATUS    RESTARTS   AGE
nginx-7c8fc894b-m6cw9   1/1       Running   0          1m

$ kubectl exec nginx-7c8fc894b-m6cw9 -it -- date
Sun Jun 24 16:35:31 KST 2018
```

Pod의 로그 또한 KST 시간을 기준으로 조회되는 것을 확인할 수 있습니다.

```
$ kubectl logs -f nginx-7c8fc894b-m6cw9
127.0.0.1 - - [24/Jun/2018:16:37:04 +0900] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.181 Safari/537.36" "-"
```
