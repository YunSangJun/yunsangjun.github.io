---
layout: post
title:  "Helm chart를 사용하여 Jenkins를 컨테이너로 배포하기"
author: 윤상준
date: 2018-05-26
categories: helm
---

이 페이지는 Helm chart를 사용하여 Jenkins를 컨테이너로 배포하는 방법에 대해 설명합니다.
자세한 내용은 [Jenkins Helm Chart](https://github.com/kubernetes/charts/tree/master/stable/jenkins)에서 참고하세요.

## 준비 사항

아래 가이드를 참고하여 Helm Client를 설치합니다.

[Helm 설치하기](/blog/helm/2018/05/27/installing-helm.html)

## 빠른 설치

아래 명령을 실행하면 Kubernetes 클러스터에 Jenkins가 배포됩니다.

```
$ helm install --name jenkins stable/jenkins
NAME:   jenkins
...
NOTES:
1. Get your 'admin' user password by running:
  printf $(kubectl get secret --namespace default jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);echo
2. Get the Jenkins URL to visit by running these commands in the same shell:
  NOTE: It may take a few minutes for the LoadBalancer IP to be available.
        You can watch the status of by running 'kubectl get svc --namespace default -w jenkins'
  ...
  echo http://$SERVICE_IP:8080/login

3. Login with the password from step 1 and the username: admin

For more information on running Jenkins on Kubernetes, visit:
https://cloud.google.com/solutions/jenkins-on-container-engine
```

정상적으로 설치가 완료되었다면 `NOTES`의 가이드를 참고하여 Jenkins 대시보드에 접속할 수 있습니다.

* 참고 : 기본 설정을 사용하므로 클러스터 환경에 따라 설치 중 에러가 발생할 수 있습니다.
이런 경우 아래 가이드를 따라 사용자 설정 방식으로 배포 할 수 있습니다.


## 사용자 정의 설치

### 설정

사용자의 Kubernetes 클러스터 환경에 맞게 Jenkins의 서비스, 스토리지 등의 설정을 변경할 수 있습니다.
아래 가이드를 참고하여 사용자의 환경에 맞도록 `values.yaml` 파일을 작성합니다.

#### Jenkins admin 사용자 암호

Jenkins admin의 초기 사용자 암호입니다. 기본 설정은 랜덤한 패스워드입니다.

```
<values.yaml>
Master:
  AdminPassword: ExamplePassword
```

#### 서비스

서비스는 Jenkins 대시보드를 접속을 어떤 방식으로 제공할지에 대한 설정입니다.
기본 설정은 LoadBalancer 방식입니다. 사용자의 클러스터의 환경에 동적으로 제공되는 LoadBalancer 서비스가 있다면 사용 가능합니다.
LoadBalancer 서비스가 없다면 아래와 같이 NodePort 또는 Ingress 방식을 사용할 수 있습니다.

- NodePort

  NodePort를 사용하면 Worker Node의 Public IP를 통하여 서비스에 접근 할 수 있습니다.
  ServiceType을 NodePort로 설정하고 NodePort에 30000~32767사이의 포트 번호를 입력합니다.
  포트 번호를 입력하지 않으면 30000~32767사이의 숫자에서 랜덤하게 선택됩니다.

  접속 URI 예시 : http://WORKER_NODE_PUBLIC_IP:3xxxx

  ```
  <values.yaml>
  Master:
    ServiceType: NodePort
    NodePort: 30000-32767
  ```

- Ingress

  Ingress를 사용하면 Ingress Controller의 Public IP를 통하여 서비스에 접근 할 수 있습니다.
  ServiceType을 ClusterIP로 설정하고 HostName에 원하는 도메인을 입력합니다.(이 때 도메인이 Ingress Controller의 Public IP와 연결되어야 합니다.)
  SSL 접속을 지원하려면 TLS 설정의 secretName과 host명을 입력합니다.

  Ingress TLS Secret 생성 방법은 아래 가이드를 참고하세요.

  [Ingress TLS Secret 생성하기](/blog/kubernetes/2018/05/27/how-to-create-ingress-secret.html)

  ```
  <values.yaml>
  Master:
    ServiceType: ClusterIP
    HostName: jenkins.example.com
    Ingress:
      TLS:
        - secretName: example-tls
          hosts:
            - jenkins.example.com
  ```

#### 스토리지

Jenkins는 영구적으로 저장해야 하는 데이터를 `/var/jenkins_home`에 저장합니다.
이 데이터를 영구적으로 저장하기 위해서는 `/var/jenkins_home` 폴더를 PV(Persistent Volume)에 mount 해야합니다.
사용자의 Cluster에서 동적으로 PV를 생성 및 관리해주는 기능이 있다면 별도의 설정이 필요없습니다.
GKE, AWS, AKS 등의 환경에서는 Storage Class를 정의하면 PV를 생성하고 관리합니다.
또한 미리 구성된 PVC(PersistentVolumeClaim)를 사용할 수 있습니다.

- Storage Class 사용

  Storage Class를 사용하여 동적으로 PV를 생성합니다.
  Storage Class 명은 아래와 같이 확인할 수 있습니다.

  ```
  $ kubectl get storageclass
  NAME                       PROVISIONER         AGE
  default                    ibm.io/ibmc-file    41d
  ibmc-block-bronze          ibm.io/ibmc-block   3d
  ibmc-file-bronze           ibm.io/ibmc-file    26d
  ...
  ```

  원하는 Storage Class 명을 입력합니다. Storage의 크기를 원하는 값으로 설정합니다.

  ```
  <values.yaml>
  Persistence:
    StorageClass: "default"
    Size: 8Gi
  ```

- 사전 구성된 PVC 사용

  미리 구성된 PVC(PersistentVolumeClaim)를 사용합니다.
  미리 구성된 PVC 명은 아래와 같이 확인할 수 있습니다.

  ```
  $ kubectl get pvc
  NAME          STATUS    VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS        AGE
  jenkins-pvc   Bound     pvc-47aec304-5e4e-11e8-915d-82c09a00d8d2   20Gi       RWO            ibmc-block-bronze   3d
  ```

  미리 구성된 PVC 명을 입력합니다.

  ```
  <values.yaml>
  Persistence:
    ExistingClaim: "jenkins-pvc"
  ```

### 설치

이제 아래 명령을 실행하여 사용자 정의에 맞게 Jenkins를 설치합니다.

```
$ helm install --name jenkins -f values.yaml stable/jenkins
```
