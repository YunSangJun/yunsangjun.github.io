---
layout: post
title:  "Terraform workspace 사용하기"
author: sj
date: 2020-11-10
categories: terraform
tags:
- terraform
- iac
- devops
- cloud
---

Terrform을 사용해 인프라 자원을 관리하다가 문득 이런 의문점이 생겼습니다.
만약 개발/운영 환경과 같이 일부 속성만 다른 경우에도 terraform 파일을 두가지 버전으로 관리해야하나?
언뜻 생각하기에도 비효율적인거 같습니다.

Terraform 문서를 찾아보니 [module](https://www.terraform.io/docs/configuration/modules.html)을
사용하면 공통적으로 사용하는 리소스를 만들고 이를 재활용할 수 있는거 같습니다.
또 Terraform best practices를 찾아보면 
[medium-terraform](https://github.com/antonbabenko/terraform-best-practices/tree/master/examples/medium-terraform) 
예제 코드에서 대략적인 내용을 확인할 수 있습니다.

이 [저장소](https://github.com/antonbabenko/terraform-best-practices/tree/master/examples)의 예제를 보면
small, medium, large 이렇게 리소스 규모에 따라 다른 예제가 있습니다.
[small terraform](https://github.com/antonbabenko/terraform-best-practices/tree/master/examples/small-terraform)인 
경우에는 굳이 module을 사용하지 않는 것을 확인할 수 있습니다. 

그래서 module을 사용할만큼 리소스가 많지 않은 경우 개발/운영계의 Terraform 파일을 하나의 버전으로 관리하면서
일부 속성만 바꿔서 사용할 수 있는 방법을 고민해봤습니다.

Terraform에는 "workspace"라는 개념이 있습니다.
workspace에는 상태를 저장하는 "terraform.tfstate" 파일이 생성되는데 workspace를 여러개 만들면 
각각의 상태정보를 관리할 수 있습니다.

이어지는 내용에서 개발/운영계 workspace를 각각 생성하고 
하나의 Terraform 파일을 사용해 개발/운영 용도의 GKE cluster를 각각 생성해보겠습니다.

## 준비하기

[Getting started Terraform with GKE](/terraform/2020/11/09/getting-started-terraform.html)에서 사용한 
[Terraform 파일 저장소](https://github.com/YunSangJun/terraform-resources.git)를 다운로드하세요.

다음 과정에서 이 파일을 사용하겠습니다.

## 개발계 workspace 생성하기

### 1.workspace 목록 조회

위에서 다운로드한 저장소의 "gcp/gke" 디렉토리 이동한 뒤 Terrafrom 초기화를 합니다.

```
$ cd terraform-resources/gcp/gke
$ terraform init
```

먼저 현재 workspace 목록을 조회해보겠습니다.
"terraform init"을 통해 초기화를 한 경우 "default" workspace만 존재합니다.

```
$ terraform workspace list
* default
```

### 2.workspace 생성

개발계 용도의 "my-dev" workspace를 생성해보겠습니다.
"my-dev"가 생성되고 switch 했다는 메세지가 출력됩니다.

```
$ terraform workspace new my-dev
Created and switched to workspace "my-dev"!
```

Workspace 목록을 다시 조회해보겠습니다.
현재 선택된 workspace가 "*"로 표시된것을 확인할 수 있습니다.

```
$ terraform workspace list
default
* my-dev
```

이번에는 디렉토리를 살펴보겠습니다.
"terraform.tfstate.d" 디렉토리가 생성되고 하위에 "my-dev" 디렉토리가 보입니다. Workspace 생성시 상응하는 디렉토리가 생성된다는 것을 알 수 있습니다.

```
$ ls -all
...
-rw-r--r--  1 ...  110524863   851 ... terraform.tfstate
drwxr-xr-x  4 ...  110524863   128 ... terraform.tfstate.d

$ ls -all terraform.tfstate.d
...
drwxr-xr-x  4 ...  110524863  128 ... my-dev
```

### 3.자원 생성

이제 개발계 용도로 사용할 GKE cluster를 생성해보겠습니다.
개발계의 service account, project, region 정보를 환경변수에 선언합니다.
그리고 "terraform apply" 명령을 실행하고 위에서 설정한 환경변수를 Terraform 변수에 할당합니다.

```
$ export GOOGLE_APPLICATION_CREDENTIALS=~/.gcp/gke-service-account-dev.json
$ export PROJECT_ID="my-dev"
$ export REGION="asia-northeast3"

$ terraform apply \
    -var "project_id=$PROJECT_ID" \
    -var "region=$REGION" \
    -var "gke_num_nodes=1"
...

Outputs:
  + kubernetes_cluster_name = "my-dev-gke"
  + region                  = "asia-northeast3"

Apply complete! Resources: 4 added, 0 changed, 0 destroyed.
```

생성이 완료되면 "node-pools"의 정보를 조회합니다. Machine type이 기본값인 "e2-medium"으로 생성된것을 확인할 수 있습니다.

```
$ gcloud container node-pools list \
  --cluster=$PROJECT_ID-gke \
  --project=$PROJECT_ID \
  --region=$REGION
NAME                     MACHINE_TYPE  DISK_SIZE_GB  NODE_VERSION
my-dev-gke-node-pool     e2-medium     100           1.16.15-gke.4300
```

### 4.terraform state 확인

이번에는 생성한 자원에 대한 terraform state를 확인해보겠습니다.
"my-dev" 디렉토리 하위에 "terraform.tfstate" 파일이 생성된 것이 보입니다.

```
$ ls -all terraform.tfstate.d/my-dev
-rw-r--r--  1 ...  110524863  12092 ... terraform.tfstate
```

이 파일 내용을 확인해보면 개발계 GKE cluster 정보가 기록되어 있는 것을 확인할 수 있습니다.

```
$ cat terraform.tfstate.d/my-dev/terraform.tfstate |more
{
...
"outputs": {
    "kubernetes_cluster_name": {
    "value": "my-dev-gke",
    "type": "string"
    },
...
```

## 운영계 workspace 생성하기

### 1.workspace 생성

운영계 용도의 "my-prod" workspace를 생성하겠습니다.

```
$ terraform workspace new my-prod
Created and switched to workspace "my-prod"!

$ terraform workspace list
default
my-dev
* my-prod
```

### 2.자원 생성

이제 운영계 용도로 사용할 GKE cluster를 생성해보겠습니다.
운영계의 service account, project, region 정보를 환경변수에 선언합니다.
그리고 "terraform apply" 명령을 실행하고 위에서 설정한 환경변수를 Terraform 변수에 할당합니다.
추가로 운영계는 machine type을 "e2-standard-2"로 다르게 설정했습니다.

```
$ export GOOGLE_APPLICATION_CREDENTIALS=~/.gcp/gke-service-account-prod.json
$ export PROJECT_ID="my-prod"
$ export REGION="asia-northeast3"

$ terraform apply \
    -var "project_id=$PROJECT_ID" \
    -var "region=$REGION" \
    -var "gke_num_nodes=1" \
    -var "gke_machine_type=e2-standard-2"
...

Outputs:
  + kubernetes_cluster_name = "my-prod-gke"
  + region                  = "asia-northeast3"

Apply complete! Resources: 4 added, 0 changed, 0 destroyed.
```

생성이 완료되면 "node-pools"의 정보를 조회합니다. Machine type이 "e2-standard-2"으로 생성된것을 확인할 수 있습니다.

```
$ gcloud container node-pools list \
  --cluster=$PROJECT_ID-gke \
  --project=$PROJECT_ID \
  --region=$REGION
NAME                     MACHINE_TYPE   DISK_SIZE_GB  NODE_VERSION
my-prod-gke-node-pool    e2-standard-2  100           1.16.15-gke.4300
```

### 3.terraform state 확인

"my-prod" 디렉토리 하위에 "terraform.tfstate" 파일이 생성되었고 이 파일 내용을 확인해보면 운영계 GKE cluster 정보가 기록되어 있는 것을 확인할 수 있습니다.

```
$ cat terraform.tfstate.d/my-prod/terraform.tfstate |more
{
...
"outputs": {
    "kubernetes_cluster_name": {
    "value": "my-prod-gke",
    "type": "string"
    },
...
```

## 자원 정리하기

자원을 정리할때도 개발/운영 각각의 workspace로 전환해 destroy 명령을 실행하면됩니다.

먼저 개발계 자원을 정리하겠습니다.

```
$ terraform workspace select my-dev

$ terraform workspace list
  default
* my-dev
  my-prod

$ export GOOGLE_APPLICATION_CREDENTIALS=~/.gcp/gke-service-account-dev.json
$ export PROJECT_ID="my-dev"

$ terraform destroy \
    -var "project_id=$PROJECT_ID" \
    -var "region=$REGION"
```

다음으로 운영계 자원을 정리합니다.

```
$ terraform workspace select my-prod

$ terraform workspace list
  default
  my-dev
* my-prod

$ export GOOGLE_APPLICATION_CREDENTIALS=~/.gcp/gke-service-account-prod.json
$ export PROJECT_ID="my-prod"

$ terraform destroy \
    -var "project_id=$PROJECT_ID" \
    -var "region=$REGION"
```

추가로 "GOOGLE_APPLICATION_CREDENTIALS, PROJECT_ID" 등의 환경변수를 alias 설정해두면 workspace 전환을 좀 더 편리하게 할 수 있습니다.

```
$ cat ~/.gcp/dev.rc
GOOGLE_APPLICATION_CREDENTIALS=~/.gcp/gke-service-account-dev.json
PROJECT_ID=my-dev
REGION="asia-northeast3"

$ cat ~/.gcp/prod.rc
GOOGLE_APPLICATION_CREDENTIALS=~/.gcp/gke-service-account-prod.json
PROJECT_ID=my-prod
REGION="asia-northeast3"

$ alias gcpdev="source ~/.gcp/dev.rc"
$ alias gcpprod="source ~/.gcp/prod.rc"
```

