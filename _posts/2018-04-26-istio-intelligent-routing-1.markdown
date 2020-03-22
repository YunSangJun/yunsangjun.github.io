---
layout: post
title:  "Istio Intelligent Routing #1 콘텐츠 기반 라우팅"
author: sj
date: 2018-04-26
categories: istio
tags:
- istio
- servicemesh
- microservice
- envoy
- loadbalancing
- canary
- abtest
- kubernetes
---

이 문서는 가중치 및 HTTP 헤더를 기반으로 동적 요청 라우팅을 구성하는 방법을 보여줍니다.

## 사전 준비

1. Kubernetes에 Istio 설치하기

    [Kubernetes에 Istio 설치하기](/blog/istio/2018/04/26/deploying-istio-on-kubernetes.html)를 참고하여 Kubernetes에 Istio 설치합니다.<br />

2. 마이크로서비스 샘플앱(BookInfo) 배포하기

    [마이크로서비스 샘플앱(BookInfo) 배포하기](/blog/istio/2018/04/26/deploying-bookinfo-on-kubernetes.html)를 참고하여 Kubernetes에 BookInfo 애플리케이션을 배포합니다.<br />


## 콘텐츠 기반 라우팅

1. 모든 마이크로서비스에 대해 기본버전을 v1으로 적용합니다.

    ```
    $ istioctl create -f samples/bookinfo/kube/route-rule-all-v1.yaml
    ```

    참고: istioctl은 kubectl로 대체할 수 있습니다. 하지만 kubectl은 현재 유효성 검사를 하지 않습니다.

    아래 명령어를 통해 route rule을 조회할 수 있습니다.
    ```
    $ istioctl get routerules -o yaml
    apiVersion: config.istio.io/v1alpha2
    kind: RouteRule
    metadata:
      name: details-default
      namespace: default
      ...
    spec:
      destination:
        name: details
      precedence: 1
      route:
      - labels:
          version: v1
    ---
    apiVersion: config.istio.io/v1alpha2
    kind: RouteRule
    metadata:
      name: productpage-default
      namespace: default
      ...
    spec:
      destination:
        name: productpage
      precedence: 1
      route:
      - labels:
          version: v1
    ---
    apiVersion: config.istio.io/v1alpha2
    kind: RouteRule
    metadata:
      name: ratings-default
      namespace: default
      ...
    spec:
      destination:
        name: ratings
      precedence: 1
      route:
      - labels:
          version: v1
    ---
    apiVersion: config.istio.io/v1alpha2
    kind: RouteRule
    metadata:
      name: reviews-default
      namespace: default
      ...
    spec:
      destination:
        name: reviews
      precedence: 1
      route:
      - labels:
          version: v1
    ---
    ```

2. BookInfo app 접속

    http://$GATEWAY_ADDRESS/productpage <br />
    Bookinfo application의 `productpage`를 볼수 있습니다.<br />
    `reviews:v1` 인스턴스 에는 rating stars가 없으므로 표시되지 않습니다.<br />

    ![](/assets/images/istio_intel_routing_contents_norating.png)

3. 특정 user로 접속

    아래 명령어를 실행하면 user `jason`이 `reviews:v2` 인스턴스로 접속되도록 설정할 수 있습니다.<br />
    ```
    $ istioctl create -f samples/bookinfo/kube/route-rule-reviews-test-v2.yaml
    ```

    설정 확인
    ```
    $ istioctl get routerule reviews-test-v2 -o yaml
    apiVersion: config.istio.io/v1alpha2
    kind: RouteRule
    metadata:
      creationTimestamp: null
      name: reviews-test-v2
      namespace: bookinfo
      resourceVersion: "517883"
    spec:
      destination:
        name: reviews
      match:
        request:
          headers:
            cookie:
              regex: ^(.*?;)?(user=jason)(;.*)?$
      precedence: 2
      route:
      - labels:
          version: v2
    ---
    ```

4. productpage web page에서 user `jason`으로 접속

    이제 ratings (1-5 stars)를 볼 수 있습니다. 로그인하지 않으면 `reviews:v1` 인스턴스로 접속됩니다.<br />

    ![](/assets/images/istio_intel_routing_contents_rating.png)

## 참고 자료
https://istio.io/docs/tasks/traffic-management/request-routing.html

## 다음 포스트
[Istio Intelligent Routing #2 오류 주입](/blog/istio/2018/05/02/istio-intelligent-routing-2.html)
