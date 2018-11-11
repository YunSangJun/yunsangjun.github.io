---
layout: post
title:  "Cronjob 사용하기"
author: 윤상준
date: 2018-06-25
categories: kubernetes
tags:
- kubernetes
- container
- cronjob
---

이 페이지는 Kubernetes 상에서 CronJob을 사용하는 방법에 대해 설명합니다.

Unix 계열의 서버 환경에서 [Cron](https://ko.wikipedia.org/wiki/Cron)을 사용해 정기적인 배치 작업을 수행하는데 활용합니다.

하지만 Kubernetes 환경에서는 서버(Pod)가 언제든지 삭제되고 다시 생성 될 수 있습니다.
이 경우 설정한 Cron 설정도 초기화됩니다.

Kubernetes 상에서는 CronJob을 활용하여 정기적인 배치 작업에 활용할 수 있습니다.
CronJob은 실행 시점에만 Pod를 생성해 자원을 사용하므로 매우 효율적입니다.

## CronJob 배포하기

샘플 CronJob을 Kubernetes 클러스터에 배포합니다.

```
$ vi cronjob.yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: sample-cron
spec:
  schedule: '*/35 * * * *'
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: busybox
            image: busybox
            imagePullPolicy: IfNotPresent
            args:
            - /bin/sh
            - -c
            - date; echo "CronJob has been excuted successfully."
          restartPolicy: OnFailure

$ kubectl apply -f cronjob.yaml
cronjob "sample-cron" created
```

CronJob에서는 Job을 생성하기 위한 jobTemplate spec 정의가 필요합니다.
또한 Job에는 Pod를 생성하기 위한 Pod spec 정의가 필요합니다.

CronJob에서는 [Unix standard crontab format](https://en.wikipedia.org/wiki/Cron#Overview)으로 schedule을 정의합니다.

- 첫 번째 값은 분을 의미합니다.(0~59 사이)
- 두 번째 값은 시간을 의미합니다.(0~23 사이)
- 세 번째 값은 일을 의미합니다.(1~31 사이)
- 네 번째 값은 월을 의미합니다.(1~12 사이)
- 다섯 번째 값은 주를 의미합니다.(0~6 사이)

스케줄에는 `*` wildcard를 사용할 수 있습니다.
`*/35 * * * *`는 매달, 매일 35분에 반복적으로 CronJob을 실행한다는 의미입니다.

## CronJob 확인하기

아래 명령어를 실행하여 등록한 CronJob을 확인할 수 있습니다.

```
$ kubectl get cronjob
NAME          SCHEDULE       SUSPEND   ACTIVE    LAST SCHEDULE   AGE
sample-cron   */35 * * * *   False     0         <none>          4s
```

Schedule에 설정한 시간이 되면 아래와 같이 Job이 실행됩니다.

```
$ kubectl get jobs
NAME                     DESIRED   SUCCESSFUL   AGE
sample-cron-1529850900   1         1            24s
```

Job이 실행되면 Pod를 생성하고 정해진 작업을 수행합니다.
Job이 생성한 Pod는 아래와 같이 조회할 수 있습니다.

```
$ kubectl get po -a
NAME                           READY     STATUS      RESTARTS   AGE
sample-cron-1529850900-5hfzn   0/1       Completed   0          41s
```

Pod의 로그를 조회해 수행한 작업 내용을 확인할 수 있습니다.

```
$ kubectl logs -f sample-cron-1529850900-5hfzn
Sun Jun 24 14:35:06 UTC 2018
CronJob has been excuted successfully.
```
