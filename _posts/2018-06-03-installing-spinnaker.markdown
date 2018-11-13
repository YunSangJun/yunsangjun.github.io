---
layout: post
title:  "Spinnaker 설치하기"
author: 윤상준
date: 2018-06-03
categories: spinnaker
tags:
- spinnaker
- kubernetes
- cicd
- devops
---

이 페이지는 Spinnaker를 설치하는 방법에 대해 설명합니다.

자세한 내용은 Spinnaker 공식 가이드 문서의 [Set up Spinnaker](https://www.spinnaker.io/setup/)을 참고하세요.

## 1.Halyard 설치하기

Halyard는 배포 구성 작성 및 유효성 검사, Spinnaker의 마이크로 서비스 배포 및 업데이트를 포함하여 Spinnaker 배포의 수명주기를 관리합니다.
운영가능한 Spinnaker를 설치 및 업데이트하기 위해 Halyard가 필요합니다. Halyard없이 Spinnaker를 설치할 수는 있지만 권장하지 않습니다.

### Debian/Ubuntu and macOS 환경

Halyard는 아래 환경을 지원합니다.

- Ubuntu 14.04 or 16.04 (Ubuntu 16.04 requires Spinnaker 1.6.0 or later)
- Debian 8 or 9
- macOS (tested on 10.13 High Sierra only)

1. Halyard 최신버전 다운로드

    - For Debian/Ubuntu:

      ```
      $ curl -O https://raw.githubusercontent.com/spinnaker/halyard/master/install/debian/InstallHalyard.sh
      ```

    - For macOS:

      ```
      $ curl -O https://raw.githubusercontent.com/spinnaker/halyard/master/install/macos/InstallHalyard.sh
      ```

2. 설치

    설치 중에 자동 완성 여부 및 bash RC 경로 물어보면 엔터키 입력(기본값으로 설정)

    ```
    $ sudo bash InstallHalyard.sh
    ...
    Would you like to configure halyard to use bash auto-completion? [default=Y]:
    Where is your bash RC? [default=/home/withdtlabs/.bashrc]:
    ```

3. 설치 확인

    ```
    $ hal -v
    ```

4. 자동 완성 설정

    ```
    $ . ~/.bashrc
    ```

### Docker 환경

1. [Docker CE](https://docs.docker.com/engine/installation/)를 설치

2. 로컬 환경에 Halyard 설정 디렉토리를 생성

    ```
    $ mkdir ~/.hal
    ```

3. Halyard Docker 컨테이너 실행

    아래 명령을 실행하여 Halyard Docker 컨테이너를 생성합니다. 이 명령은 Halyard 설정 및 kubeconfig 디렉토리를 마운트합니다.

    ```
    $ docker run -p 8084:8084 -p 9000:9000 \
        --name halyard --rm \
        -v ~/.hal:/home/spinnaker/.hal \
        -v ~/.kube:/home/spinnaker/.kube \
        -it \
        gcr.io/spinnaker-marketplace/halyard:stable
    ```

4. Halyard에 접속

    아래 명령을 실행하여 Halyard Docker 컨테이너에 접속합니다.

    ```
    $ docker exec -it halyard bash
    ```

5. 자동 완성 설정

    아래 명령을 실행하여 자동완성을 설정합니다.

    ```
    $ source <(hal --print-bash-completion)
    ```



`hal` 명령어에 대한 자세한 내용은 [Halyard command Reference](https://www.spinnaker.io/reference/halyard/commands)를 참고하세요.



## 2.Cloud 공급자 선택하기

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

- kubectl

    Spinnaker는 kubectl을 사용하여 모든 API 액세스를 관리합니다. Spinnaker와 함께 설치됩니다.

#### 계정 추가

이제 계정을 추가합니다. 먼저, 공급자가 활성화 되어 있는지 확인합니다.

```
$ hal config provider kubernetes enable
```

그리고 계정을 추가합니다.

```
$ hal config provider kubernetes account add my-k8s-v2-account \
    --provider-version v2 \
    --context $(kubectl config current-context)

$ hal config features edit --artifacts true
```

#### 추가 설정

추가 설정에 대한 내용은 [Halyard Reference](https://www.spinnaker.io/reference/halyard/commands#hal-config-provider-kubernetes-account-add)를 참고하세요.

## 3.설치 환경 선택하기

Halyard가 Spinnaker를 어떤 방식으로 설치할지 선택합니다.

- 분산 설치

Halyard가 Spinnaker’s 마이크로서비스를 분산 설치합니다. 운영 환경으로 설치 시 권장합니다.

- 로컬 설치

하나의 머신에 설치됩니다. 소규모 배포에 적합합니다.

- github에서 설치

Spinnaker 프로젝트에 기여하는 개발자에게 적합합니다.

이 페이지에서는 분산 설치 방식으로 진행해보겠습니다.

### 분산 설치

분산 설치는 리소스가 많은 개발 조직 및 Spinnaker 업데이트 중 다운 타임을 없어야 하는 경우에 적합합니다.

Spinnaker는 원격 클라우드에 배포되며 각 마이크로 서비스는 독립적으로 배포됩니다. Halyard는 Spinnaker 마이크로 서비스를 무중단으로 업데이트합니다.

Spinnaker를 설치하기 위해 `4 cores`와 `8GB of RAM`을 권장합니다.

아래 명령을 실행합니다. `$ACCOUNT`는 공급자 선택에서 설정한 생성한 계정입니다.

```
$ hal config deploy edit --type distributed --account-name $ACCOUNT
```

## 4.스토리지 선택하기

Spinnaker에는 애플리케이션 설정 및 파이프 라인 설정을 유지하기 위해 외부 저장소가 필요합니다.

Spinnaker는 아래와 같은 스토리지를 지원합니다. 어떤 옵션을 선택해도 Cloud 공급자 선택에 영향을 미치지 않습니다.
예를 들어, Google Cloud Storage를 저장소 소스로 사용할 수 있지만 여전히 Microsoft Azure에 배포 할 수 있습니다.

- Azure Storage
- Google Cloud Storage
- Minio
- Redis
- S3

<p class="warning-title">경고</p>
<p class="warning-content">
Redis는 운영환경에서는 권장하지 않습니다.
</p>

### Google Cloud Storage

#### 사전 준비

- Google Cloud Platform(GCP) 프로젝트 생성

- [gcloud](https://cloud.google.com/sdk/downloads) 설치

#### Credential 다운로드

Spinnaker는 GCP에 인증하기 위해 `roles/storage.admin` 역할이 활성화된 서비스 계정이 필요합니다.

계정이 없다면 아래와 같이 생성합니다.

```
SERVICE_ACCOUNT_NAME=spinnaker-gcs-account
SERVICE_ACCOUNT_DEST=~/.gcp/gcs-account.json

gcloud iam service-accounts create \
    $SERVICE_ACCOUNT_NAME \
    --display-name $SERVICE_ACCOUNT_NAME

SA_EMAIL=$(gcloud iam service-accounts list \
    --filter="displayName:$SERVICE_ACCOUNT_NAME" \
    --format='value(email)')

PROJECT=$(gcloud info --format='value(config.project)')

gcloud projects add-iam-policy-binding $PROJECT \
    --role roles/storage.admin --member serviceAccount:$SA_EMAIL

mkdir -p $(dirname $SERVICE_ACCOUNT_DEST)

gcloud iam service-accounts keys create $SERVICE_ACCOUNT_DEST \
    --iam-account $SA_EMAIL
```

#### 스토리지 설정

Halyard는 버킷을 설정하지 않았거나 설정한 버킷이 없으면 새로운 bucket을 생성합니다.

아래는 기본 버킷 설정 예제입니다.

```
PROJECT=$(gcloud info --format='value(config.project)')
# see https://cloud.google.com/storage/docs/bucket-locations
BUCKET_LOCATION=us
SERVICE_ACCOUNT_DEST=# 전 단계의 설정 참고

hal config storage gcs edit --project $PROJECT \
    --bucket-location $BUCKET_LOCATION \
    --json-path $SERVICE_ACCOUNT_DEST
```

스토리지를 GCS로 설정합니다.

```
hal config storage edit --type gcs
```

### Minio

Minio의 데이터를 잃으면 Spinnaker 애플리케이션 메타 데이터 및 파이프 라인이 모두 손실됩니다.

Minio는 Self 호스팅 할 수있는 S3 호환 Object Storage입니다. Spinnaker 데이터를 호스팅하기 위해 클라우드 제공 업체에 의존하고 싶지 않을 때 권장되는 영구 저장소 솔루션입니다.

#### 사전 준비

1. [Minio 홈페이지](https://www.minio.io/)에 있는 가이드를 따라 Minio를 설치합니다.

2. S3에 Bucket을 생성합니다. 이 Bucket 명을 아래 스토리지 설정에서 사용합니다.

#### 스토리지 설정

Minio가 버전 객체를 지원하지 않으므로 Spinnaker에서 버전 객체를 비활성화합니다.

```
$ vi ~/.hal/$DEPLOYMENT/profiles/front50-local.yml:
spinnaker.s3.versioning: false
```

$DEPLOYMENT 일반적으로 `default`입니다.

아래 명령을 실행해 S3를 저장소 유형으로 선택합니다.

```
$ export ENDPOINT=S3_ENDPOINT
$ export MINIO_ACCESS_KEY=S3_ACCESS_KEY_ID

$ hal config storage s3 edit --endpoint $ENDPOINT \
    --access-key-id $MINIO_ACCESS_KEY \
    --bucket BUCKET_NAME
    --secret-access-key
  Your AWS Secret Key.:

$ hal config storage edit --type s3
```

## 5.Spinnaker 설치

### 버전 선택

설치 가능한 버전을 조회합니다.

```
$ hal version list
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
$ hal config version edit --version $VERSION
```

### 설치

이제 Spinnaker를 배포합니다.

```
$ hal deploy apply
```

### Spinnaker 대시보드 접속

아래 명령어를 실행합니다.

```
$ hal deploy connect
```

웹 브라우저에서 `localhost:9000` 에 접속합니다.

## Trouble Shooting

- `Service: Amazon S3; Status Code: 400; Error Code: InvalidLocationConstraint`

    아래와 같은 에러 발생 시 S3에서 bucket을 수동으로 생성 후 halyard config에서 새로 생성한 bucket명으로 수정한 후 다시 배포해봅니다.

    ```
    Caused by: org.springframework.beans.BeanInstantiationException: Failed to instantiate [com.netflix.spinnaker.front50.model.S3StorageService]: Factory method 's3StorageService' threw exception; nested exception is com.amazonaws.services.s3.model.AmazonS3Exception: Container storage location with specified provisioning code not available (Service: Amazon S3; Status Code: 400; Error Code: InvalidLocationConstraint; Request ID: d08c26ee-34ba-4e62-83bc-5d9a26b23947; S3 Extended Request ID: null), S3 Extended Request ID: null
    ```

    ```
    $ vi ~/.hal/config
    - name: default
      persistentStorage:
        s3:
          bucket: example-bucket

    $ hal deploy apply      
    ```

## 6.Spinnaker 업그레이드

### 버전 조회

현재 버전(1.8.5)과 업그레이드 가능한 버전을 조회합니다.

```
$ hal version list
+ Get current deployment
  Success
+ Get Spinnaker version
  Success
+ Get released versions
  Success
+ You are on version "1.8.5", and the following are available:
 - 1.7.8 (Ozark):
   Changelog: https://gist.github.com/spinnaker-release/75f98544672a4fc490d451c14688318e
   Published: Wed Aug 29 19:09:57 UTC 2018
   (Requires Halyard >= 1.0.0)
 - 1.8.7 (Dark):
   Changelog: https://gist.github.com/spinnaker-release/ebb5e45e84de5b4381b422e3c8679b5a
   Published: Fri Sep 28 17:58:52 UTC 2018
   (Requires Halyard >= 1.0.0)
 - 1.9.5 (Bright):
   Changelog: https://gist.github.com/spinnaker-release/d24a2c737db49dda644169cf5fe6d56e
   Published: Mon Oct 01 17:15:37 UTC 2018
   (Requires Halyard >= 1.0.0)
```

### 버전 설정

업그레이드 하기 원하는 버전을 설정합니다.

```
$ hal config version edit --version 1.9.5
+ Get current deployment
  Success
+ Edit Spinnaker version
  Success
Problems in halconfig:
        oauthScopes: []
- WARNING There is a newer version of Halyard available (1.11.0),
  please update when possible
? Run 'sudo update-halyard' to upgrade
+ Spinnaker has been configured to update/install version "1.9.5".
  Deploy this version of Spinnaker with `hal deploy apply`.
```

### 적용

변경사항을 반영합니다.

```
$ hal deploy apply
+ Get current deployment
  Success
+ Prep deployment
  Success
Problems in halconfig:
- WARNING There is a newer version of Halyard available (1.11.0),
  please update when possible
? Run 'sudo update-halyard' to upgrade
+ Preparation complete... deploying Spinnaker
+ Get current deployment
  Success
+ Apply deployment
  Success
+ Run `hal deploy connect` to connect to Spinnaker.
```
