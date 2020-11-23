---
layout: post
title:  "Terraform backend 사용하기"
author: sj
date: 2020-11-22
categories: terraform
tags:
- terraform
- iac
- devops
- cloud
---

Terrform은 state 파일에 인프라 자원의 상태를 저장합니다. 이 state 파일은 기본적으로 local에 저장됩니다.
혼자서 인프라 자원을 관리하는 경우 특별히 문제가 없지만 여러명이 협업을 하는 경우에는 어떨까요?

먼저 state 파일을 형상 관리하고 이를 각 사용자의 local 환경에 다운로드 받아 사용하는 경우가 있을 수 있습니다.
이 경우 누군가 변경한 사항이 있는데 이를 업데이트하지 않고 자신이 변경한 사항을 바로 인프라 환경에 적용해 문제가 발생할 수 있습니다.

또는 공동으로 사용하는 서버(예: bastion)에 state 파일을 관리하는 경우가 있을 수 있습니다.
이 경우 여러 사용자가 동시에 state 파일을 변경할 경우 문제가 발생할 수 있습니다.

이렇게 여러명이 Terraform state 파일을 공유해야하는 경우
[Backend](https://www.terraform.io/docs/backends/index.html) 기능을 사용하는 것이 좋습니다.
Backend를 사용할 경우 state 파일을 원격 저장소에 안전하게 보관할 수 있고, state 파일을 사용 시 lock을 설정해 충돌을 방지할 수 있습니다.

[Backend 유형](https://www.terraform.io/docs/backends/types/index.html)은 여러가지 방식을 지원합니다.
Backend를 지정하지 않는 경우 기본으로 local 환경에 state 파일이 생성됩니다.
이 문서에서는 GCS(Google Cloud Storage)를 backend로 설정하고 인프라 자원을 생성해보겠습니다.
그리고 local 환경에서 state 파일을 관리할 때와 어떤 차이점이 있는지 비교해보겠습니다.

## 준비하기

### 1.Service account 생성하기

[Getting started Terraform with GKE](/terraform/2020/11/09/getting-started-terraform.html)에서 사용한
Service account에 "Cloud Storage - Storage Admin" 역할을 추가하세요.

### 2.Bucket 생성하기

Terraform의 backend로 사용할 bucket을 생성합니다. 이 문서에서는 GCS(Google Cloud Storage)를 backend로 사용하겠습니다.
[Bucket 생성하기](https://cloud.google.com/storage/docs/creating-buckets)를 참고하여 bucket을 생성하세요.

그리고 생성한 bucket에 [객체 버전 관리](https://cloud.google.com/storage/docs/using-object-versioning#gsutil)
를 활성화하는 것을 권장합니다. 버전 관리를하면 Terraform state 파일을 쉽게 복구할 수 있습니다.

### 3.예제코드 다운로드하기

[Getting started Terraform with GKE](/terraform/2020/11/09/getting-started-terraform.html)에서 사용한 
[Terraform 파일 저장소](https://github.com/YunSangJun/terraform-resources.git)를 다운로드하세요.

다음 과정에서 이 파일을 사용하겠습니다.

## Terraform backend 선언하기

위에서 다운로드한 저장소의 "gcp/gke" 디렉토리 이동한 뒤 main.tf 파일을 아래와 같이 편집합니다.

backend의 type에는 "gcs"를 입력합니다.
bucket에는 위에서 생성한 bucket명을 입력합니다. 
prefix에는 gke라고 입력하겠습니다. prefix는 Terraform state 파일의 상위 폴더입니다.
아래와 같이 설정을 하면 Terraform state 파일이 my-tfstate/gke/default.tfstate로 생성됩니다.

```
$ cd terraform-resources/gcp/gke

$ vi main.tf
provider "google" {
  ...
}

terraform {
  backend "gcs" {
    bucket  = "my-tfstate
    prefix  = "gke"
  }
}
...
```

[GCS backend](https://www.terraform.io/docs/backends/types/gcs.html)에서 추가적인 설정을 확인할 수 있습니다.

## Workspace 초기화하기

자원을 생성하기 전 Terrafrom workspace를 초기화하겠습니다.
Workspace 초기화 방법은 backend를 local로 사용할 때와 동일합니다.
다른 점은 backend가 "gcs"로 설정되었다는 메세지가 출력된다는 점입니다.
기본 workspace는 동일하게 "default"로 생성됩니다.

```
$ terraform init
Initializing the backend...

Successfully configured the backend "gcs"! Terraform will automatically
use this backend unless the backend configuration changes.

Initializing provider plugins...
...

Terraform has been successfully initialized!
...

$ terraform workspace list
* default
```

Terraform state 파일을 조회해보면 앞에서 설정한 gcs backend의 상태가 local 환경에 저장되어 있는것을 확인할 수 있습니다.
이 state 파일은 단순히 backend의 상태를 저장하며 생성하는 자원의 상태와는 관계가 없습니다.
따라서 삭제되어도 초기화를 통해 다시 생성하면 됩니다.
이는 바꿔말하면 초기화만 해주면 어떤 환경에서든 동일한 상태정보를 유지할 수 있다는 뜻이기도 합니다.

```
$ cat .terraform/terraform.tfstate 
{
    ...
    "backend": {
        "type": "gcs",
        "config": {
            "bucket": "my-tfstate",
            "prefix": "gke",
            ...
        },
        ...
    },
    ...
}
```

다음으로 GCS의 bucket을 조회해보겠습니다. 
prefix인 gke 폴더 하위에 default workspace에 상응하는 defualt.tfstate 파일이 생성된 것을 확인할 수 있습니다.
파일명 뒤의 숫자는 이 파일의 버전명입니다. Bucket에 버전 관리를 활성화한 경우 파일이 변경될 때마다 다른 버전명으로 파일이 새로 생성됩니다.
파일을 복사해서 내용을 조회해보면 아직 생성한 자원이 없으므로 outputs, resources가 빈 값입니다.

```
$ gsutil ls -a gs://my-tfstate/gke
gs://my-tfstate/gke/default.tfstate#1606049661300720

$ gsutil copy gs://my-tfstate/gke/default.tfstate#1606049661300720 ./default.tfstate

$ cat ./default.tfstate
{
  "version": 4,
  "terraform_version": "0.13.5",
  "serial": 0,
  "lineage": "...,
  "outputs": {},
  "resources": []
}
```

## 자원 생성하기

이제 GKE cluster를 생성해보겠습니다.
자원 생성 방법은 backend를 local로 사용할 때와 동일합니다.

service account, project, region 정보를 환경변수에 선언합니다.
그리고 "terraform apply" 명령을 실행하고 설정한 환경변수를 Terraform 변수에 할당합니다.

```
$ export GOOGLE_APPLICATION_CREDENTIALS=~/.gcp/gke-service-account.json
$ export PROJECT_ID="my-project"
$ export REGION="asia-northeast3"

$ terraform apply \
    -var "project_id=$PROJECT_ID" \
    -var "region=$REGION" \
    -var "gke_num_nodes=1"
```

자원이 생성되는 동안 bucket을 다시 조회해보겠습니다.
"default.tflock"이라는 파일이 생성되었습니다. 
이 파일은 Terraform state 파일을 변경하는 동안 다른 사람이 동시에 변경할 수 없도록 lock을 설정하기 위한 용도입니다.

```
$ gsutil ls -a gs://my-tfstate-295114/gke
gs://my-tfstate/gke/default.tfstate#1606049661300720
gs://my-tfstate/gke/default.tflock#1606056208610188

$ gsutil copy gs://my-tfstate/gke/default.tflock#1606056208610188 ./default.tflock

$ cat ./default.tflock
{"ID":"...","Operation":"OperationTypeApply","Info":"","Who":"...","Version":"0.13.5","Created":"...","Path":"gs://my-tfstate/gke/default.tflock"}
```

자원 생성이 완료되면 bucket을 다시 조회합니다.
outputs, resources에 생성한 GKE cluster의 상태가 저장되어 있는것을 확인할 수 있습니다.

```
$ gsutil copy gs://my-tfstate/gke/default.tfstate#1606049661300720 ./default.tfstate

$ cat ./default.tfstate
  ...
  "outputs": {
    "kubernetes_cluster_name": {
      "value": "my-project-gke",
      "type": "string"
    }
  },
  "resources": [
  ...
```

## 자원 정리하기

사용하지 않는 자원을 정리하겠습니다.
자원 삭제 방법은 backend를 local로 사용할 때와 동일합니다.

```
$ terraform destroy \
    -var "project_id=$PROJECT_ID" \
    -var "region=$REGION"
```

