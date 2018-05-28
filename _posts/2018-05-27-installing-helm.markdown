---
layout: post
title:  "Helm 설치하기"
author: 윤상준
date: 2018-05-27
categories: helm
---

이 페이지는 Helm을 설치하는 방법에 대해 설명합니다.
자세한 내용은 [Installing Helm](https://docs.helm.sh/using_helm/#installing-helm)을 참고하세요.

## Helm client 바이너리 다운로드

1. [Helm release](https://github.com/kubernetes/helm/releases) 바이너리를 다운로드합니다.

2. 바이너리 압축 해제합니다.

  ```
  tar -zxvf helm-vX.X.X-OS_NAME-amd64.tgz
  ```

3. Helm 바이너리를 원하는 위치로 이동시킵니다.

  ```
  mv OS_NAME-amd64/helm /usr/local/bin/helm
  ```

## Tiller 설치

Helm의 서버 부분 인 Tiller는 일반적으로 Kubernetes 클러스터 내부에서 실행됩니다. 그러나 개발을 위해 로컬로 실행될 수도 있고 원격 Kubernetes 클러스터와 통신하도록 구성 될 수도 있습니다.

Tiller를 클러스터에 설치하는 가장 쉬운 방법은 `helm init`를 실행하는 것입니다.

`helm init` 실행 후 Tiller가 running 상태가 될 때까지 대기합니다.

```
$ kubectl get pods --namespace kube-system |grep tiller
tiller-deploy-7ccf99cd64-pd7c2        1/1       Running   0          3d
```

Helm client와 server의 버전을 확인합니다.
Server의 버전이 Client의 버전과 같거나 높아야 합니다.

```
$ helm version
Client: &version.Version{SemVer:"v2.9.0", GitCommit:"xxx", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.9.1", GitCommit:"xxx", GitTreeState:"clean"}
```

Server를 제외한 Client만 설치하고 싶은 경우 아래와 같이 실행합니다.

```
$ helm init --client-only
```

이제 Helm을 활용해 chart를 Kubernetes에 배포할 수 있습니다.
