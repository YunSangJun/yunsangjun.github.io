---
layout: post
title:  "Kubernetes에 Istio 설치하기"
author: 윤상준
date: 2018-04-26 20:00:39 +0900
categories: istio
---

이 문서를 따라 Kubernetes에 Istio 설치 할 수 있습니다.

# Istio 다운로드 및 설치

1. Istio release 다운로드

    아래 Istio github에서 원하는 버전의 release download
    <br />
    https://github.com/istio/istio/releases

2. 설치파일 추출

    ```
    $ tar zxvf istio-x.x.x-xxx.tar.gz
    ```

3. istioctl binary를 /usr/local/bin으로 이동

    ```
    $ cd istio-x.x.x
    $ mv bin/istioctl /usr/local/bin/
    ```

4. Kubernetes에 Istio 설치

    Istio는 `istio-system` namespace에 배포됩니다.

    ```
    //without tls
    $ kubectl apply -f install/kubernetes/istio.yaml

    //with tls
    $ kubectl apply -f install/kubernetes/istio-auth.yaml
    ```

5. Istio svc 확인

    ```
    $ kubectl get svc -n istio-system
    NAME            TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)                                                             AGE
    istio-ingress   LoadBalancer   172.21.x.x       169.x.x.x       80:30493/TCP,443:31629/TCP                                          20h
    istio-mixer     ClusterIP      172.21.x.x       <none>          9091/TCP,15004/TCP,9093/TCP,9094/TCP,9102/TCP,9125/UDP,42422/TCP    20h
    istio-pilot     ClusterIP      172.21.x.x       <none>          15003/TCP,15005/TCP,15007/TCP,15010/TCP,8080/TCP,9093/TCP,443/TCP   20h
    ```

6. Istio pod 확인

    ```
    $ kubectl get pods -n istio-system
    NAME                             READY     STATUS    RESTARTS   AGE
    istio-ca-86f55cc46f-5pcj6        1/1       Running   0          20h
    istio-ingress-5bb556fcbf-n99cr   1/1       Running   0          20h
    istio-mixer-86f5df6997-rtld9     3/3       Running   0          20h
    istio-pilot-67d6ddbdf6-svnfp     2/2       Running   0          20h
    ```

# 다음 포스트
[마이크로서비스 샘플앱(BookInfo) 배포하기](/blog/istio/2018/04/26/deploying-bookinfo-on-kubernetes.html)
