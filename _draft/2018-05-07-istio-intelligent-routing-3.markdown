---
layout: post
title:  "Istio Intelligent Routing #3 가중치 기반 라우팅"
author: sj
date: 2018-05-07
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

이 문서는 서비스의 이전 버전에서 새 버전으로 트래픽을 점진적으로 마이그레이션하는 방법을 보여줍니다.

## 사전 준비

1. Kubernetes에 Istio 설치하기

    [Kubernetes에 Istio 설치하기](/istio/2018/04/26/deploying-istio-on-kubernetes.html)를 참고하여 Kubernetes에 Istio 설치합니다.<br />

2. 마이크로서비스 샘플앱(BookInfo) 배포하기

    [마이크로서비스 샘플앱(BookInfo) 배포하기](/istio/2018/04/26/deploying-bookinfo-on-kubernetes.html)를 참고하여 Kubernetes에 BookInfo 애플리케이션을 배포합니다.<br />


## 가중치 기반 라우팅

1. 모든 마이크로서비스 대해 기본 버전을 v1으로 적용합니다.

    ```
    $ istioctl create -f samples/bookinfo/kube/route-rule-all-v1.yaml
    ```
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

2. 브라우저에서 `http://$GATEWAY_ADDRESS/productpage` 페이지를 열어 v1이 reviews 서비스의 기본 버전인지 확인합니다.

    Bookinfo 애플리케이션의 productpage가 표시되어야합니다. v1은 ratings 서비스에 액세스하지 않으므로 productpage에 별표가 표시되지 않습니다.

    참고 : 이전에 콘텐츠 기반 라우팅 작업을 실행 한 경우 테스트 사용자 `jason`으로 로그 아웃하거나 생성 된 테스트 규칙을 삭제해야 합니다.

    ```
    $ istioctl delete routerule reviews-test-v2
    ```

3. 먼저 트래픽을 reviews:v1과 reviews:v3에 50:50 비율로 전송합니다.

    ```
    $ istioctl replace -f samples/bookinfo/kube/route-rule-reviews-50-v3.yaml
    ```

    아래 명령어를 통해 route rule을 조회할 수 있습니다.
    ```
    $ istioctl get routerule reviews-default -o yaml
    apiVersion: config.istio.io/v1alpha2
    kind: RouteRule
    metadata:
      name: reviews-default
      namespace: default
    spec:
      destination:
        name: reviews
      precedence: 1
      route:
      - labels:
          version: v1
        weight: 50
      - labels:
          version: v3
        weight: 50
    ```

    브라우저에서 productpage를 새로 고침하면 빨간색 별표가 약 50%의 비율로 표시됩니다.

    ![](/assets/images/istio_intel_routing_contents_red_rating.png)

4. 이번에는 트래픽을 reviews:v1과 reviews:v3에 20:80 비율로 전송합니다.

    아래와 같이 route rule을 수정합니다.

    참고 : istioctl CLI에는 edit 명령어가 없으므로 kubectl 명령어를 사용합니다.

    ```
    $ kubectl edit routerule reviews-default
    apiVersion: config.istio.io/v1alpha2
    kind: RouteRule
    metadata:
      name: reviews-default
      namespace: default
    spec:
      destination:
        name: reviews
      precedence: 1
      route:
      - labels:
          version: v1
        weight: 20
      - labels:
          version: v3
        weight: 80
    ```

    브라우저에서 productpage를 새로 고침하면 빨간색 별표가 약 80%의 비율로 표시됩니다.

5. reviews 마이크로서비스의 버전 v3가 안정적이라고 판단되면 트래픽의 100%를 reviews:v3에 라우팅 할 수 있습니다.

    ```
    $ istioctl replace -f samples/bookinfo/kube/route-rule-reviews-v3.yaml
    ```

    이제 아무 사용자로 제품 페이지에 로그인 할 수 있으며 항상 빨간색 별표가 표시됩니다.

## 이해하기
이 예제에서는 Istio의 가중치 기반 라우팅 기능을 사용하여 `reviews` 서비스로의 트래픽을 이전 버전에서 새 버전으로 마이그레이션했습니다. 이는 인스턴스 scaling을 사용하여 트래픽을 관리하는 컨테이너 오케스트레이션 플랫폼의 배포 기능을 사용하는 버전 마이그레이션과 매우 다릅니다. Istio를 사용하면 `reviews` 서비스의 두 버전간의 트래픽 분산에 영향을 미치지 않고 독립적으로 확장 및 축소 할 수 있습니다.

## 참고 자료
https://istio.io/docs/tasks/traffic-management/traffic-shifting.html
