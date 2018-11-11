---
layout: post
title:  "Istio Intelligent Routing #2 오류 주입"
author: 윤상준
date: 2018-05-02
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

이 문서는 지연을 주입하고 애플리케이션의 복원력을 테스트하는 방법을 보여줍니다.

## 사전 준비

1. Kubernetes에 Istio 설치하기

    [Kubernetes에 Istio 설치하기](/blog/istio/2018/04/26/deploying-istio-on-kubernetes.html)를 참고하여 Kubernetes에 Istio 설치합니다.<br />

2. 마이크로서비스 샘플앱(BookInfo) 배포하기

    [마이크로서비스 샘플앱(BookInfo) 배포하기](/blog/istio/2018/04/26/deploying-bookinfo-on-kubernetes.html)를 참고하여 Kubernetes에 BookInfo 애플리케이션을 배포합니다.<br />

3. 애플리케이션의 버전 라우팅 초기화하기

    아래 명령을 실행하여 버전 라우팅을 초기화합니다. 이미 생성한 라우팅이 있으면 `create` 대신 `replace` 명령을 사용합니다.
    ```
    istioctl create -f samples/bookinfo/kube/route-rule-all-v1.yaml
    istioctl create -f samples/bookinfo/kube/route-rule-reviews-test-v2.yaml
    ```

## HTTP 지연을 사용한 오류 주입
Bookinfo 마이크로서비스의 복원력을 테스트하기 위해, `reviews:v2`와 `ratings` 마이크로서비스 사이에 7s의 지연을 주입합니다. `reviews:v2` 서비스는 `ratings` 서비스로의 요청에 대해 10s의 timeout을 갖기 때문에, 그 요청에 에러가 발생하지 않을 것으로 예상할 수 있습니다.

1. 사용자 `jason`의 트래픽을 지연시키는 오류 주입 규칙을 생성합니다.

    ```
    $ istioctl create -f samples/bookinfo/kube/route-rule-ratings-test-delay.yaml
    ```

    생성한 규칙 확인
    ```
    $ istioctl get routerule ratings-test-delay -o yaml
    apiVersion: config.istio.io/v1alpha2
    kind: RouteRule
    metadata:
      name: ratings-test-delay
      namespace: default
      ...
    spec:
      destination:
        name: ratings
      httpFault:
        delay:
          fixedDelay: 7.000s
          percent: 100
      match:
        request:
          headers:
            cookie:
              regex: ^(.*?;)?(user=jason)(;.*)?$
      precedence: 2
      route:
      - labels:
          version: v1
    ```

    모든 pods에 규칙이 전파되도록 몇 초간 대기합니다.

2. 애플리케이션의 동작 확인

    사용자 `jason`으로 로그인합니다. 애플리케이션의 프론트 페이지가 지연을 올바르게 처리하도록 설정되어 있으면, 약 7초 내로 로드될 것으로 예상됩니다.
    웹 페이지 응답 시간을 보려면, IE, Chrome 또는 Firefox (일반적으로 `Ctrl + Shift + I` 또는 `Alt + Cmd + I` 키 조합)의 개발자 도구 메뉴를 열고 네트워크 탭을 클릭 한 다음 `productpage`를 다시 로드하십시오.<br>
    약 6 초 후에 웹 페이지가 로드됩니다. `reviews` 섹션에 <b>"죄송합니다. 현재 이 책에 대한 제품 리뷰를 사용할 수 없습니다."</b>라고 표시 될 것입니다.

    ![](/blog/assets/images/istio_fault_injection_delay_error.png)

## 이해하기
`reviews` 서비스에 에러가 난 이유는 Bookinfo 애플리케이션에 버그가 있기 때문입니다. `productpage`와 `reviews` 서비스 간의 timeout(3s + 1 retry = 6s total)은 `reviews`와 `ratings` 서비스 사이의 timeout(10s)보다 작습니다. 이런 버그는 서로 다른 팀이 마이크로서비스를 독립적으로 개발하는 환경에서 발생할 수 있습니다. Istio의 <b>오류 주입</b> 규칙은 사용자에게 영향을 주지 않고 이런 예외를 식별하는 데 도움을 줍니다.<br>

이 예제에서는 에러가 사용자 `jason`에게만 영향을 주도록 제한하고 있습니다. 다른 사용자로 로그인하면 지연이 발생하지 않습니다.<br>

버그 수정: 일반적으로 `productpage` timeout 늘리거나 `reviews`와 `ratings` 서비스 간의 timeout을 줄여 문제를 해결합니다.<br>
아래와 같이 지연을 2.8초로 변경하여 문제가 해결되는지 확인합니다.

```
$ kubectl edit routerule ratings-test-delay
apiVersion: config.istio.io/v1alpha2
kind: RouteRule
metadata:
  creationTimestamp: null
  name: ratings-test-delay
  ...
spec:
  destination:
    name: ratings
  httpFault:
    delay:
      fixedDelay: 2.800s
      percent: 100
  match:
    request:
      headers:
        cookie:
          regex: ^(.*?;)?(user=jason)(;.*)?$
  precedence: 2
  route:
  - labels:
      version: v1
---
```

![](/blog/assets/images/istio_fault_injection_delay_fixed.png)

## HTTP 중단을 사용한 오류 주입
다른 복원력 테스트 방법으로써 HTTP 중단을 소개합니다. 지연을 통한 방법과 달리 페이지가 즉시 로드되고 'ratings 서비스 사용할 수 없음'이라는 메시지가 표시됩니다.

1. 지연을 사용한 오류 주입 규칙을 삭제합니다.

    ```
    $ istioctl delete -f samples/bookinfo/kube/route-rule-ratings-test-delay.yaml
    ```

2. 사용자 `jason`에게 HTTP 중단을 보내기 위한 오류 주입 규칙을 생성합니다.

    ```
    $ istioctl create -f samples/bookinfo/kube/route-rule-ratings-test-abort.yaml
    ```

    생성한 규칙 확인
    ```
    $ istioctl get routerules ratings-test-abort -o yaml
    apiVersion: config.istio.io/v1alpha2
    kind: RouteRule
    metadata:
      name: ratings-test-abort
      namespace: default
      ...
    spec:
      destination:
        name: ratings
      httpFault:
        abort:
          httpStatus: 500
          percent: 100
      match:
        request:
          headers:
            cookie:
              regex: ^(.*?;)?(user=jason)(;.*)?$
      precedence: 2
      route:
      - labels:
          version: v1
    ```

3. 애플리케이션의 동작 확인
사용자 `jason`으로 로그인합니다. 규칙이 모든 pods에 전파되면 "ratings 서비스 사용할 수 없음"이라는 메시지와 함께 페이지가 표시됩니다. 사용자 `jason`에서 로그 아웃하면 `productpage` 페이지에 ratings v1이 정상적으로 표시됩니다.

    HTTP 중단을 사용한 오류 주입 상태
    ![](/blog/assets/images/istio_fault_injection_aborted.png)

    ratings v1이 정상적으로 표시 상태
    ![](/blog/assets/images/istio_fault_injection_not_aborted.png)

## 참고 자료
https://istio.io/docs/tasks/traffic-management/fault-injection.html

## 다음 포스트
[Istio Intelligent Routing #3 가중치 기반 라우팅](/blog/istio/2018/05/07/istio-intelligent-routing-3.html)
