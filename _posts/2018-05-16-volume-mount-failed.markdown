---
layout: post
title:  "MountVolume.SetUp failed for volume pvc-xxx"
author: 윤상준
date: 2018-05-16 21:20:05 +0900
categories: kubernetes
---

## Issue

Pod가 STATUS가 ContainerCreating 상태임.
```
$ kubectl get pod xxx
NAME   READY    STATUS              RESTARTS   AGE       IP        
xxx    0/1      ContainerCreating   0          1m        <none>  
```

Describe 명령어로 상태를 확인.
Pod가 mount 하려는 pvc의 device가 이미 mount 되어 있음. Pod를 비정상 종료되면서 umount가 안된것으로 보임.
```
$ kubectl describe pod xxx
...
Events:
  Type     Reason                 Age   From                     Message
  ----     ------                 ----  ----                     -------
  Normal   Scheduled              1m    default-scheduler        Successfully assigned xxx to 10.178.218.181
  Normal   SuccessfulMountVolume  1m    kubelet, 10.178.218.181  MountVolume.SetUp succeeded for volume "default-token-z84t8"
  Warning  FailedMount            13s   kubelet, 10.178.218.181  MountVolume.SetUp failed for volume "pvc-xxx" : mount command failed, status: Failure, reason: Error while mounting the volume &errors.errorString{s:"RWO check has failed. DevicePath /var/lib/kubelet/plugins/kubernetes.io/flexvolume/ibm/ibmc-block/mounts/pvc-xxx is already mounted on mountpath /var/lib/kubelet/pods/496dc423-4da7-11e8-915d-82c09a00d8d2/volumes/ibm~ibmc-block/pvc-xxx "}
```

## Solution

1. 해당 Pod가 실행중인 Node에 접속

```
$ kubectl get pod xxx -o wide
NAME  READY     STATUS              RESTARTS   AGE       IP        NODE
xxx   0/1       ContainerCreating   0          1m        <none>    10.178.218.181

$ ssh user@10.178.218.181
```

2. 해당 Pod가 mount 하려는 device를 찾아서 umount

```
$ df -h |grep xxx
/dev/mapper/xxx  20G  232M  19G  2%   /var/lib/kubelet/plugins/kubernetes.io/flexvolume/ibm/ibmc-block/mounts/pvc-xxx

$ umount -v /dev/mapper/xxx
/dev/mapper/xxx umounted...
```

3. 해당 Pod 삭제해 재시작하면 정상적으로 mount 됨

```
$ kubectl delete pod xxx

$ kubectl describe pod xxx
...
Events:
  Type    Reason                 Age   From                     Message
  ----    ------                 ----  ----                     -------
  Normal  Scheduled              43m   default-scheduler        Successfully assigned xxx to 10.178.218.181
  Normal  SuccessfulMountVolume  43m   kubelet, 10.178.218.181  MountVolume.SetUp succeeded for volume "default-token-z84t8"
  Normal  SuccessfulMountVolume  43m   kubelet, 10.178.218.181  MountVolume.SetUp succeeded for volume "pvc-xxx"
  ...
  Normal  Created                42m   kubelet, 10.178.218.181  Created container
  Normal  Started                42m   kubelet, 10.178.218.181  Started container

```
