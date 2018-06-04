---
layout: post
title:  "Spinnaker 설치하기"
author: 윤상준
date: 2019-06-03
categories: spinnaker
---

이 페이지는 Spinnaker를 설치하는 방법에 대해 설명합니다.
자세한 내용은 [Set up Spinnaker](https://www.spinnaker.io/setup/)을 참고하세요.

## Halyard 설치하기

Halyard는 배포 구성 작성 및 유효성 검사, Spinnaker의 마이크로 서비스 배포 및 업데이트를 포함하여 Spinnaker 배포의 수명주기를 관리합니다.
운영가능한 Spinnaker를 설치 및 업데이트하기 위해 Halyard가 필요합니다. Halyard없이 Spinnaker를 설치할 수는 있지만 권장하지 않습니다.

1. [Docker CE](https://docs.docker.com/engine/installation/)를 설치

2. 설치 환경에 로컬 Halyard 설정 디렉토리를 생성

    ```
    mkdir ~/.hal
    ```

3. Docker 컨테이너에서 Halyard를 실행

    아래 명령을 실행하여 Halyard Docker 컨테이너를 생성합니다. 이 명령은 Halyard 설정 디렉토리를 마운트합니다.

    ```
    docker run -p 8084:8084 -p 9000:9000 \
        --name halyard --rm \
        -v ~/.hal:/home/spinnaker/.hal \
        -v ~/.kube:/home/spinnaker/.kube \
        -it \
        gcr.io/spinnaker-marketplace/halyard:stable
    ```

4. Halyard에 접속

    아래 명령을 실행하여 Halyard Docker 컨테이너에 접속합니다.

    ```
    docker exec -it halyard bash
    ```

5. 자동 완성

    아래 명령을 실행하여 자동완성을 설정합니다.

    ```
    source <(hal --print-bash-completion)
    ```

    `hal` 명령어에 대한 자세한 내용은 [Halyard command Reference](https://www.spinnaker.io/reference/halyard/commands)를 참고하세요.

## Cloud 공급자 선택하기

Spinnaker를 통해 애플리케이션을 배포할 Cloud 공급자를 선택합니다.

아래와 같은 Cloud 공급자를 지원합니다.

- App Engine
- Amazon Web Services
- Amazon Web Services - ECS
- Azure
- DC/OS
- Google Compute Engine
- Kubernetes (legacy)
- Kubernetes V2 (manifest based)
- Openstack
- Oracle

이 페이지에서는 `Kubernetes V2`를 선택해 Spinnaker를 설치해보겠습니다.

### Kubernetes V2 공급자

`Kubernetes V2`는 Spinnaker 1.6의 알파 기능을 포함하고 있으므로 `Kubernetes (legacy)`에 비해 불안정할 수 있습니다.

#### 계정

`Kubernetes V2`의 경우 Spinnaker 계정이 Kubernetes Cluster에 대해 인증 할 수 있는 자격 증명에 매핑됩니다. V1 공급자와 달리 V2에서는 Docker Registry Accounts가 필요하지 않습니다.

#### 사전 준비

아래 두 가지 사항이 필요합니다.

- kubeconfig

    kubeconfig를 사용하면 Spinnaker가 관리 할 것으로 예상되는 모든 리소스에 대한 읽기/쓰기 권한을 가질 수 있습니다. Kubernetes 클러스터 관리자에게 요청할 수 있습니다.

    이 페이지에서는 Docker 컨테이너에서 Halyard를 실행시 로컬환경의 `~/.kube` 폴더를 Docker 컨테이너의 `/home/spinnaker/.kube`에 mount 했습니다.
    로컬환경의 `~/.kube` 폴더 하위에 kubeconfig를 생성하면됩니다.

- kubectl

    Spinnaker는 kubectl을 사용하여 모든 API 액세스를 관리합니다. Spinnaker와 함께 설치됩니다.

#### 계정 추가

이제 계정을 추가합니다. 먼저, 공급자가 활성화 되어 있는지 확인합니다.

```
hal config provider kubernetes enable
```

그리고 계정을 추가합니다.

```
hal config provider kubernetes account add my-k8s-v2-account \
    --provider-version v2 \
    --context $(kubectl config current-context)
    
hal config features edit --artifacts true
```

#### 추가 설정

추가 설정에 대한 내용은 [Halyard Reference](https://www.spinnaker.io/reference/halyard/commands#hal-config-provider-kubernetes-account-add)를 참고하세요.

## 설치환경 선택하기

Halyard가 Spinnaker를 어디에 설치할지 선택합니다.

- 분산 설치

Halyard가 Spinnaker’s 마이크로서비스를 분산 설치합니다. 운영환경으로 설치 시 권장합니다.

- 로컬 설치

하나의 머신에 설치됩니다. 소규모 배포에 적합합니다.
 of Debian packages Spinnaker is deployed on a single machine. This is good for smaller deployments.

- github에서 설치

Spinnaker 프로젝트에 기여하는 개발자에게 적합합니다.

이 페이지에서는 분산 설치 방식으로 진행해보겠습니다.

### 분산 설치

분산 설치는 리소스가 많은 개발 조직 및 Spinnaker 업데이트 중 다운 타임을 없어야 하는 경우에 적합합니다.

Spinnaker는 원격 클라우드에 배포되며 각 마이크로 서비스는 독립적으로 배포됩니다. Halyard는 Spinnaker 마이크로 서비스를 무중단으로 업데이트합니다.

Spinnaker를 설치하기 위해 `4 cores`와 `8GB of RAM`을 권장합니다.

아래 명령을 실행합니다. `$ACCOUNT`는 공급자 선택에서 설정한 생성한 계정입니다.

```
hal config deploy edit --type distributed --account-name $ACCOUNT
```

## 스토리지 선택하기

Spinnaker에는 애플리케이션 설정 및 파이프 라인 설정을 유지하기 위해 외부 저장소가 필요합니다.

Spinnaker는 아래와 같은 스토리지를 지원합니다. 어떤 옵션을 선택해도 Cloud 공급자 선택에 영향을 미치지 않습니다.
예를 들어, Google Cloud Storage를 저장소 소스로 사용할 수 있지만 여전히 Microsoft Azure에 배포 할 수 있습니다.

- Azure Storage
- Google Cloud Storage
- Minio
- Redis
- S3

참고: Redis는 운영환경에서는 권장하지 않습니다.

이 페이지에서는 Minio를 사용해서 Self 호스팅하는 S3와 연동하는 방식으로 진행해보겠습니다.

### Minio

Minio의 데이터를 잃으면 Spinnaker 애플리케이션 메타 데이터 및 파이프 라인이 모두 손실됩니다.

Minio는 Self 호스팅 할 수있는 S3 호환 Object Storage입니다. Spinnaker 데이터를 호스팅하기 위해 클라우드 제공 업체에 의존하고 싶지 않을 때 권장되는 영구 저장소 솔루션입니다.

#### 사전 준비

[Minio 홈페이지](https://www.minio.io/)에 있는 가이드를 따라 Minio를 설치합니다. 

#### 스토리지 설정

```
$ export ENDPOINT=S3_ENDPOINT
$ export MINIO_ACCESS_KEY=S3_ACCESS_KEY_ID

$ hal config storage s3 edit --endpoint $ENDPOINT \
    --access-key-id $MINIO_ACCESS_KEY \
    --secret-access-key 
Your AWS Secret Key.: 
+ Get current deployment
  Success
+ Get persistent store
  Success
Generated bucket name: spin-e994b7ef-2c80-48bf-935c-d1e784417dc2
+ Edit persistent store
  Success
Problems in default.persistentStorage:
- WARNING Your deployment will most likely fail until you configure
  and enable a persistent store.
+ Successfully edited persistent store "s3". 

$ hal config storage edit --type s3
+ Get current deployment
  Success
+ Get persistent storage settings
  Success
+ Edit persistent storage settings
  Success
+ Successfully edited persistent storage.
```

## Spinnaker 설치

설치 가능한 버전을 조회합니다.

```
hal version list
+ Get current deployment
  Success
+ Get Spinnaker version
  Success
+ Get released versions
  Success
+ You are on version "", and the following are available:
 - 1.5.4 (Atypical):
   Changelog: https://gist.github.com/spinnaker-release/6b9fd632caeaefd32246074998af8498
   Published: Wed Jan 10 18:46:49 UTC 2018
   (Requires Halyard >= 0.40.0)
 - 1.6.1 (GLOW):
   Changelog: https://gist.github.com/spinnaker-release/f1cd6232151b70492ebdcbb557a209fc
   Published: Wed Apr 04 19:20:54 UTC 2018
   (Requires Halyard >= 0.41.0)
 - 1.7.6 (Ozark):
   Changelog: https://gist.github.com/spinnaker-release/5d3af465f07eaca64f4383167877897d
   Published: Tue May 29 16:26:20 UTC 2018
   (Requires Halyard >= 1.0.0)
```

원하는 버전을 선택합니다.

```
hal config version edit --version $VERSION
```

ingress에서 사용할 도메인을 입력합니다.

```
hal config security ui edit --override-base-url http://spinnaker.zcp-dev.jp-tok.containers.mybluemix.net
hal config security api edit --override-base-url http://spinnaker-api.zcp-dev.jp-tok.containers.mybluemix.net
```

이제 Spinnaker를 배포합니다.

```
hal deploy apply
```


