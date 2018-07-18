---
layout: post
title:  "Kubernetes 스터디 #3 워크로드"
author: 윤상준
date: 2018-07-02
categories: kubernetes
---

## Pods

Pod는 Kubernetes에서 가장 작은 단위의 구성요소입니다.

Pod는 하나 또는 여러개의 애플리케이션 컨테이너, 스토리지, 네트워크 아이피를 포함합니다.

Pod는 아래와 같이 두 가지 형태로 사용할 수 있습니다.

- 단일 컨테이너

  Pod당 하나의 컨테이너를 사용하는 방식이 가장 일반적입니다.

- 여러개의 컨테이너

  컨테이너 간에 서로 밀접하게 연관되어 있거나 자원을 공유해야 하는 경우 Pod에 여러개의 컨테이너를 캡슐화 할 수 있습니다.

![](/blog/assets/images/kubernetes/kubernetes-workload-pod.png)

### Pod Replication과 Controller

Pod는 하나의 애플리케이션을 인스턴스를 실행합니다.

만약 애플리케이션을 수평적으로 확장하기 원하는 경우 여러개의 Pod를 사용해야 합니다.

Kubernetes에서는 이를 Replication라고 합니다.

복제된 Pod들은 컨트롤러(Controller)에 의해 그룹으로 관리됩니다.

이러한 이유로 Pod를 단독으로 생성하지 않고 컨트롤러를 통해 생성하는 것이 일반적입니다.

![](/blog/assets/images/kubernetes/kubernetes-workload-pod-replication-and-controller.png)

### 사이드카(멀티 컨테이너)

이 패턴은 컨테이너가 tight하게 결합되어 리소스를 공유해야 하는 경우에 사용합니다.

예를 들어 하나의 Pod에 공유 저장소를 사용하는 웹 서버 컨테이너와 외부에서 파일을 불러오는 컨테이너를 함께 실행할 수 있습니다.

이 때 외부에서 파일을 불러오는 컨테이너를 `사이드카(Sidecar)`라고 합니다.

![](/blog/assets/images/kubernetes/kubernetes-workload-pod-sidecar.png)

## Controllers

Pod를 Replication하고 관리하는 Controller는 여러가지 방식이 있습니다.
Controller별 동작 및 사용방식에 대해 살펴보겠습니다.

### ReplicationController

ReplicationController는 Replication Controller의 초기 버전입니다.
최신 버전의 Kubernetes에서는 Pod를 Replication하기 위한 Controller로 ReplicaSet을 제어하는 Deployment를 사용하는 것을 권장합니다.

ReplicationController는 지정한 수 만큼의 Pod가 실행되는지 체크합니다.
만약 특정 Pod에 에러가 나서 비정상 종료되거나 삭제되면 자동으로 새로운 Pod를 생성해 교체합니다.

Pod의 개수를 확장 또는 축소하고 싶은 경우 복제본(replicas) 개수를 조정하면 ReplicationController가 자동으로 Pod를 생성 또는 삭제합니다.

ReplicationController에 대한 자세한 내용은
[Kubernetes Docs](https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/)
를 참고하세요.

### ReplicaSet

ReplicaSet은 Replication Controller의 신규 버전입니다. ReplicationController와 마찬가지로 Pod의 생성 및 삭제를 관리합니다.

ReplicaSet과 ReplicationController 간의 유일한 차이점은 지원하는 selector 종류입니다.
ReplicaSet은 새로운 `set-based` selector를 지원하지만 ReplicationController는 `equality-based` selector만을 지원합니다.

ReplicaSets를 단독으로 사용할 수 있지만 주로 Deployment를 통해서 ReplicaSets을 사용합니다.
왜냐하면 Deployment가 ReplicaSet을 관리하기 때문입니다.

또 rolling-update 기능을 사용하는 경우 Deployment를 사용하는 것이 좋습니다.
Deployment를 통해 ReplicaSet을 관리하면 rollout, rollback 기능을 사용하는 것이 편리합니다.

ReplicaSet에 대한 자세한 내용은
[Kubernetes Docs](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)
를 참고하세요.

### Deployments



### StatefulSets

### DaemonSet

### Jobs

### CronJob
