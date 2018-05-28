---
layout: post
title:  "Istio Overview"
author: 윤상준
date: 2018-04-26
categories: istio
---

## Overview
Istio는 로드밸런싱, 서비스 대 서비스 인증, 모니터링을 통해 코드 변경없이 배포된 서비스의 네트워크를 쉽게 생성할 수 있는 방법을 제공합니다.<br />
Istio의 control plane을 사용하여 설정 및 관리되고, 마이크로서비스의 모든 통신을 가로채는 환경에 특수한 sidecar proxy를 배포하여 Istio를 서비스에 추가할 수 있습니다.<br />
Istio는 현재 Kubernetes 환경만을 지원하지만 향후 다른 환경도 지원할 예정입니다.<br />
Istio components에 대한 자세한 정보를 원하면 다른 컨셉들의 가이드를 참고하세요. <br />

## 왜 Istio를 사용하는가?
모놀리식 애플리케이션이 마이크로 서비스 아키텍처로 전환함에 따라 개발자와 운영자는 많은 문제에 직면합니다.<br />
마이크로 서비스의 네트워크(서비스 매시)의 크기와 복잡성이 커짐에 따라 이해와 관리가 어려워 질 수 있습니다.<br />
이런 어려움을 해결하기 위해서 아래와 같은 요구사항이 있습니다.<br />
```
디스커버리, 로드밸런싱, 복구, 메트릭, 모니터링, A/B 테스트, canary 배포, 속도 제한, 접근 제어, end-to-end 인증
```
<br />
Istio는 마이크로 서비스 애플리케이션의 다양한 요구 사항을 충족시킬 수있는 솔루션을 제공합니다.<br />

- 트래픽 관리
서비스간의 트래픽과 API 호출 흐름을 제어하고, 호출을 보다 안정적으로 만들고, 불리한 조건에서도 네트워크를 강하게 만듭니다.

- 관찰
서비스간의 의존성 및 트래픽간의 특성과 흐름을 파악하여 문제를 빠르게 식별할 수 있습니다.

- 정책 적용
서비스간의 상호작용에 조직 정책을 적용하고, 접근 정책 적용 및 컨슈머 사이에 균일하게 분산된 리소스를 보장합니다.<br />
정책 변경은 애플리케이션 코드 변경이 아니라 매시 설정에 의해 만들어집니다.

- 서비스 신원 및 보안
매시에서 검증가능한 신원을 가진 서비스를 제공하고, 다양한 신뢰도의 네트워크를 통해 전송되는 서비스 트래픽을 보호하는 기능을 제공합니다.

이 외에도, Istio는 다양한 배포 요구사항을 충족할 수 있도록 확장성있게 디자인되었습니다.<br />

- 플랫폼 지원
Istio는 Cloud, On-premise, Kubernetes, Mesos 등 다양한 환경에서 실행할 수 있도록 설계되었습니다.<br />
현재는 Kubernetes 환경에 중점을 두고 있지만 곧 다른 환경에서도 지원할 예정입니다.

- 통합과 커스터마이징
정책 적용 컴포넌트를 확장하고 통합하여 ACLs, 로깅, 모니터링, 할당량, 감사등의 기존 솔루션과 통합할 수 있습니다.

이런 기능들은 애플리케이션 코드, 플랫폼, 정책 간의 결합을 크게 줄입니다.<br />
애플이케이션간의 결합이 감소하면 서비스 구현을 쉽게 할 뿐 아니라, 운영자에게 서로 다른 환경 간이나 새로운 정책 구성으로의 애플리케이션 이동을 용이하게 합니다.<br />
애플리케이션은 본질적으로 이식성이 더 높아집니다.

## Architecture
Istio 서비스 매시는 논리적으로 data plane과 control plane으로 나누어집니다.<br />

Data plane은 지능형 프록시(Envoy)로 구성됩니다. 이 프록시는 사이드카로 배포되고 마이크로서비스 사이의 모든 네트워크 통신을 중재 및 제어합니다.<br />

Control plane은 트래픽을 라우팅하기 위해 프록시를 관리하고 설정합니다. 또한 런타임에 정책을 적용합니다.<br />

아래 다이어그램은 각 plane을 구성하는 다양한 컴포넌트를 보여줍니다.

![Istio Architecture](/blog/assets/images/istio_architecture.png)

### Envoy

Istio는 C++로 개발된 고성능 프록시인 Envoy 프록시의 확장 버전을 사용하여 서비스 매시의 모든 서비스에 대한 인바운드 및 아웃바운드 트래픽을 중재합니다.<br />
Istio는 아래와 같은 Envoy의 많은 내장된 기능을 사용합니다.<br />
```
dynamic service discovery, load balancing, TLS termination, HTTP/2 & gRPC proxying, circuit breakers, health checks, staged rollouts with %-based traffic split, fault injection, and rich metrics
```

Envoy는 같은 Kubernetes pod의 관련 서비스에 사이트카로 배포되고, 이를 통해 Istio는 트래픽에 대한 신호를 속성으로 추출할 수 있습니다.<br />
이 값을 Mixer에서 정책 결정을 적용하는데 사용할 수 있으며, 모니터링 시스템으로 보내져 전체 매시의 동작에 대한 정보를 제공할 수 있습니다.<br />
사이드카 프록시 모델을 사용하면 아키텍처나 코드 변경없이 기존 배포 환경에 Istio 기능을 추가할 수 있습니다.<br />

### Mixer
Mixer는 서비스 매시 전반에 접근 제어와 사용 정책을 적용하고 Envoy 프록시와 다른 서비스로부터 원격 측정 데이터를 수집하는 플랫폼 독립적인 컴포넌트입니다.<br />
프록시는 요청 레벨 속성을 추출하여 평가를 위해 Mixer로 보냅니다. 이 속성에 대한 더 많은 정보 및 정책 평가에 대한 자세한 내용은 [Mixer Configuration](https://istio.io/docs/concepts/policy-and-control/mixer-config.html)에 있습니다.<br />
Mixer는 다양한 호스트 환경 및 인프라 백앤드와 인터페이스 할 수 있는 유연한 플러그인 모델을 포함하며 Envoy 프록시와 Istio 관리 서비스를 추상화 합니다.

### Pilot
Pilot은 Envoy 사이드카에 대한 서비스 검색, 지능형 라우팅(예, A/B 테스트, canary 배포 등) 및 복원(timeouts, retries, circuit breakers 등)을 위한 트래픽 관리 기능을 제공합니다.<br />
Pilot은 트래픽 동작을 Envoy 특정 구성으로 제어하는 상위 레벨 라우팅을 변환하여 런타임의 사이드카에 전달합니다.<br />
Pilot은 플랫폼 특화 서비스 검색 메커니즘을 추상화하고 이를 data plane APIs를 준수하는 모든 사이드카에 표준 포맷으로 통합합니다.<br />
이 느슨한 결합을 통해 Istio는 트래픽 관리를 위한 동일한 운영자 인터페이스를 유지하면서도 여러 환경(예, Kubernetes, Consul/Nomad)에 실행될 수 있습니다.

### Istio-Auth
Istio-Auth는 상호 TLS, 내장 아이디, 자격 증명 관리를 사용하는 강력한 서비스 대 서비스 및 엔드 유저 인증 기능을 제공합니다.<br />
이것은 서비스 매시에서 암호화되지 않은 트래픽을 업그레이드하는데 사용 할 수 있으며, 네트워크 제어 대신 서비스 아이디에 기반하여 정책 적용을 할 수 있습니다.<br />
Istio의 향후 배포에는 세분화 된 접근 제어 및 감사 기능이 추가되어 누가 서비스, API, 리소스에 접근하는지 제어 및 모니터링 할 수 있습니다.

## 참고 자료
https://istio.io/docs/concepts/what-is-istio/overview.html

## 다음 포스트
[Kubernetes에 Istio 설치하기](/blog/istio/2018/04/26/deploying-istio-on-kubernetes.html)
