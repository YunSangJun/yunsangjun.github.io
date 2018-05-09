---
layout: post
title:  "마이크로서비스 샘플앱(BookInfo) 배포하기"
author: 윤상준
date: 2018-04-26 20:20:39 +0900
categories: istio
---

이 문서에서는 Istio 서비스 매시의 다양한 기능을 시연하는데 사용할 4개의 마이크로서비스 샘플 애플리케이션을 배포합니다.

## Overview
이 가이드에서는 온라인 서점의 카탈로그 항목과 비슷한 책에 대한 정보를 표시하는 간단한 애플리케이션을 배포합니다. 이 페이지에는 책에 대한 설명, 책 세부 정보 (ISBN, 페이지 수 등) 및 몇 권의 서평이 표시됩니다.<br>

Bookinfo 애플리케이션은 4개의 마이크로서비스로 나뉩니다.<br>

- productpage : 세부 사항을 호출하고 마이크로서비스를 검토하여 페이지를 채웁니다.

- details : 도서 정보가 들어 있습니다.

- reviews : 서평이 포함되어 있습니다. 또한 ratings 마이크로서비스를 호출합니다.

- ratings : 서평을 수반하는 서적 순위 정보가 포함됩니다.

reviews 마이크로서비스에는 3가지 버전이 있습니다.<br>

- 버전 v1은 ratings 서비스를 호출하지 않습니다.
- 버전 v2는 ratings 서비스를 호출하고 각 순위를 1 - 5 개의 검은 별로 표시합니다.
- 버전 v3는 ratings 서비스를 호출하고 각 순위를 1 ~ 5 개의 빨간색 별로 표시합니다.

애플리케이션의 end-to-end 아키텍처는 아래와 같습니다. 이 애플리케이션은 다양한 프로그래밍 언어로 작성되었습니다.<br>

![BookInfo Application Without Istio](/blog/assets/images/bookinfo_noistio.svg)

## 사전 준비

1. Kubernetes에 Istio 설치하기

    [Kubernetes에 Istio 설치하기](/blog/istio/2018/04/26/deploying-istio-on-kubernetes.html)를 참고하여 Kubernetes에 Istio 설치합니다.<br />

## BookInfo 애플리케이션
Istio와 함께 샘플을 실행하기 위해 애플리케이션을 변경할 필요는 없습니다. 대신, Istio가 활성화 된 환경에서 서비스를 구성하고 실행하면 각각의 서비스에 Envoy 사이드카가 주입됩니다. 필요한 명령과 구성은 런타임 환경에 따라 다르지만 모든 경우의 결과는 다음과 같습니다.

![BookInfo Application With Istio](/blog/assets/images/bookinfo_withistio.svg)

모든 마이크로서비스는 Envoy 사이드카와 함께 패키지화되어 수신 및 발신 요청을 가로챕니다. Istio control plane, 라우팅, 원격 측정 수집 및 애플리케이션 전체에 대한 정책 적용을 통해 외부 제어에 필요한 후크를 제공합니다.

## Kubernetes에서 BookInfo 애플리케이션 실행

1. BookInfo 애플리케이션 실행

    수동으로 사이트카 주입을 하는 경우 아래 명령어를 실행합니다.
    ```
    $ kubectl apply -f <(istioctl kube-inject -f samples/bookinfo/kube/bookinfo.yaml)
    ```

    자동 사이드카 주입 기능이 활성화되어 있는 경우 아래 명령어를 실행합니다.
    ```
    $ kubectl apply -f samples/bookinfo/kube/bookinfo.yaml
    ```

    istioctl kube-inject 명령은 애플리케이션을 배포하기 전에 bookinfo.yaml 파일을 수동으로 수정하는 데 사용됩니다.
    이 명령은 다이어그램에서 설명한 것처럼 네 개의 마이크로서비스를 시작하고 게이트웨이 ingress 자원을 생성합니다. `reviews` 서비스 v1, v2 및 v3의 3 가지 버전이 모두 시작됩니다.<br>

2. BookInfo svc 확인

    ```
    $ kubectl get svc
    NAME          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
    details       ClusterIP   172.21.67.160    <none>        9080/TCP   20h
    productpage   ClusterIP   172.21.39.218    <none>        9080/TCP   20h
    ratings       ClusterIP   172.21.201.191   <none>        9080/TCP   20h
    reviews       ClusterIP   172.21.154.60    <none>        9080/TCP   20h
    ```

3. BookInfo pod 확인

    ```
    $ kubectl get pods
    NAME                              READY     STATUS    RESTARTS   AGE
    details-v1-55496dcd64-qjqmb       2/2       Running   0          20h
    productpage-v1-586897968d-j2kwf   2/2       Running   0          20h
    ratings-v1-6d9f5df564-4kzlt       2/2       Running   0          20h
    reviews-v1-5985df7dd4-x22zl       2/2       Running   0          20h
    reviews-v2-856d5b976-vtlct        2/2       Running   0          20h
    reviews-v3-c4fbb98d8-5bv9j        2/2       Running   0          20h
    ```

4. BookInfo ingress 확인

    ```
    $  kubectl get ingress
    NAME      HOSTS     ADDRESS         PORTS     AGE
    gateway   *         169.x.x.x       80        20h
    ```

5. BookInfo 접속

    ```
    export GATEWAY_ADDRESS=169.x.x.x
    http://$GATEWAY_ADDRESS/productpage
    ```

    ![](/blog/assets/images/istio_intel_routing_contents_norating.png)

## 참고 자료
https://istio.io/docs/guides/bookinfo.html

## 다음 포스트
[Istio Intelligent Routing #1 콘텐츠 기반 라우팅](/blog/istio/2018/04/26/istio-intelligent-routing-1.html)
