---
layout: post
title:  "Getting started Terraform with GKE"
author: sj
date: 2020-11-09
categories: terraform
tags:
- terraform
- iac
- devops
- cloud
- gke
---

이전에 IBM Cloud 환경에서 Terrform을 사용해본 후 [Terraform 사용하기](/terraform/2018/06/29/terraform.html)란 글을 작성했습니다.

업무 환경이 바뀌어 한 동안 Terraform을 사용할 일이 없었는데 최근에 Kubernetes cluster를 자주 배포/삭제해야할 일이 있어 다시 사용하게 됐습니다. 거의 2년만에 사용하는 건데 다행히 문법이나 기본 틀 자체는 크게 바뀌지 않았네요.

처음에는 Terrform으로 GKE(Google Kubernetes Engine)를 배포하는 방법에 대해서만 기록하려고 했는데 작성하다보니 Terrform 초보자가 보면 좋을거 같다는 생각이 들어 "Terraform getting started"로 주제를 변경했습니다.
저도 Terraform을 아주 깊게 아는건 아니지만 초보자 입장에서 궁금했던 점들을 공부하면서 기록했으니 초보자분들이 보시면 도움이 되리라 생각합니다.
(주된 목적은 잊어 버리기전에 기록해두고 나중에 다시 보려는...)

그럼 이제부터 terraform을 사용하여 GKE를 배포하고 삭제하는 과정을 단계별로 차근차근 살펴보겠습니다.
(성격 급하신 분들이나 결론만 보고 싶으신 분들은 "TL;DR;" 섹션으로 바로 가주세요.)

## 준비하기

### 1.API 활성화하기

GKE를 생성하기 위해 아래 API를 활성화합니다.
- Compute Engine API
- Kubernetes Engine API

API를 활성화하는 방법은 [Enabling APIs](https://cloud.google.com/apis/docs/getting-started#enabling_apis) 문서를 참고하세요.

### 2.Service account 생성하기

아래 역할(Role)을 가지고 있는 Service account를 생성합니다.
- Kubernetes Engine Admin
- Compute Network Admin
- Service Account User

Service account를 생성하는 방법은 [Create a service account](https://cloud.google.com/docs/authentication/production#create_service_account) 문서를 참고하세요.

Service account를 생성 시 json 파일로 export를 합니다. 다음 과정에서 이 파일을 사용할 것입니다.

## TL;DR;(Too Long; Don't Read)

성격이 급하신 분들은 아래 코드를 실행해 GKE를 바로 배포해볼 수 있습니다.
(위에 준비하기의 과정은 수행하셔야합니다.)
```
## Clone terraform code for GKE
$ git clone https://github.com/YunSangJun/terraform-resources
$ cd terraform-resources/gcp/gke

## Initialize Terraform workspace
$ terraform init

## Set your environment
$ export GOOGLE_APPLICATION_CREDENTIALS="REPLACE_ME"
$ export PROJECT_ID="REPLACE_ME"
$ export REGION="REPLACE_ME"

## Provision GKE cluster
$ terraform apply \
    -var "project_id=$PROJECT_ID" \
    -var "region=$REGION" \
    -var "gke_num_nodes=1"
```

## Terraform 파일 작성하기

GKE를 생성하기 위한 Terraform 파일을 작성해보겠습니다.

작성하는 코드는 [Terraform 파일 저장소](https://github.com/YunSangJun/terraform-resources/tree/main/gcp/gke)에서 다운로드할 수 있습니다.

### 1.자원 정의

GKE를 생성하기 위한 자원을 Terraform 파일에 작성해보겠습니다.

먼저 "terraform-gke" 디렉토리를 생성하고 [main.tf](https://github.com/YunSangJun/terraform-resources/blob/main/gcp/gke/main.tf) 파일을 생성합니다.(main.tf 전체 코드는 링크 참조)

```
$ mkdir terraform-gke
$ cd terraform-gke

$ vi main.tf
provider {}
...

resource {}
...

```

이어서 "main.tf"에 선언할 내용을 상세히 살펴보겠습니다.

#### 1.1.Provider

먼저 provider를 선언합니다.
- provider name: google
- [provider version](https://github.com/hashicorp/terraform-provider-google/releases): "~> 3.46"
- project: var.project_id(GCP의 project 이름, "variables.tf"에 정의)
- region: var.region(GCP의 region 이름, "variables.tf"에 정의)

```
provider "google" {
  version = "~> 3.46"
  project = var.project_id
  region  = var.region
}
```

#### 1.2.Network

다음으로 GKE를 생성할 network를 선언합니다.
(기존 network에 영향을 주지 않고 테스트해보기 위해 별도로 생성)

##### 1)VPC network
- resource type: google_compute_network(생성할 resource type)
- local name: vpc(local에서 참조할 이름)
- auto_create_subnetworks: false(subnet 자동 생성 옵션)

```
resource "google_compute_network" "vpc" {
  name                    = "${var.project_id}-vpc"
  auto_create_subnetworks = "false"
}
```

##### 2)Subnet network
- ip_cidr_range: "10.10.0.0/24"(CIRD 범위)

```
resource "google_compute_subnetwork" "subnet" {
  name          = "${var.project_id}-subnet"
  region        = var.region
  network       = google_compute_network.vpc.name
  ip_cidr_range = "10.10.0.0/24"
}
```

#### 1.3.GKE cluster

이제 GKE cluster에 대해 선언합니다.

##### 1)GKE cluster
- min_master_version: "1.16"(Master의 최소 버전)
- remove_default_node_pool: true(Default node pool 삭제 여부)
- initial_node_count: 1(각 zone에 생성할 node 수)

```
resource "google_container_cluster" "primary" {
  name     = "${var.project_id}-gke"
  location = var.region
  min_master_version = "1.16"
  remove_default_node_pool = true
  initial_node_count       = 1
  network    = google_compute_network.vpc.name
  subnetwork = google_compute_subnetwork.subnet.name
}
```

<p class="tip-title">참고</p>
<p class="tip-content">
"remove_default_node_pool"는 무슨 뜻일까요?.
Cluster는 default node pool 없이 생성이 불가능합니다(일종의 template).
그래서 remove_default_node_pool 값을 "true"로 설정하면 default node pool을 작게 생성한 뒤 바로 삭제합니다.
</p>

##### 2)Separately Managed Node Pool
- node_count: var.gke_num_nodes(각 zone에 생성할 node 수, "variables.tf"에 정의)
- node_config.oauth_scopes: API 접근 권한
- node_config.metadata.disable-legacy-endpoints: true(Legacy endpoint 비활성화 여부)

```
resource "google_container_node_pool" "primary_nodes" {
  name       = "${google_container_cluster.primary.name}-node-pool"
  location   = var.region
  cluster    = google_container_cluster.primary.name
  node_count = var.gke_num_nodes
  node_config {
    oauth_scopes = [
      "https://www.googleapis.com/auth/logging.write",
      "https://www.googleapis.com/auth/monitoring",
    ]

    labels = {
      env = var.project_id
    }

    machine_type = var.gke_machine_type
    tags         = ["gke-node", "${var.project_id}-gke"]

    metadata = {
      disable-legacy-endpoints = "true"
    }
  }
}
```

<p class="tip-title">참고</p>
<p class="tip-content">
"node_count"는 각 zone에 생성할 node의 수입니다.
만약 이 값을 3으로 설정하고 region이 "asia-northeast3"라면 3개의 zone에 각각 3개의 node가 생성되어 총 9개의 node가 생성됩니다.
</p>

<p class="tip-title">참고</p>
<p class="tip-content">
GKE 1.12 버전 이상부터 "disable-legacy-endpoints" 값은 "true"가 기본으로 설정됩니다.
만약 metadata를 설정하고 "disable-legacy-endpoints"의 기본값을 선언하지 않으면, 
terraform은 이 값을 해제합니다. 
이를 방지하기 위해서 "disable-legacy-endpoints" 값을 "true"로 설정해야합니다.
</p>

### 2.변수 정의

"main.tf" 파일에서 사용하는 변수를 선언하겠습니다.

"variables.tf" 파일을 생성하고 project_id, region, gke_num_nodes, gke_machine_type 변수를 선언합니다.

default 값이 있는 경우 정의해주고, description 값에 변수에 대한 주석을 작성합니다.
이 변수는 terraform 명령 실행 시 다른 값으로 치환할 수 있습니다.

```
$ vi variables.tf
variable "project_id" {
  description = "project id"
}

variable "region" {
  description = "region"
}

variable "gke_num_nodes" {
  default     = 3
  description = "number of gke nodes"
}

variable "gke_machine_type" {
  default     = "e2-medium"
  description = "machine type of gke nodes"
}
```

### 3.결과 정의

자원을 생성한 후 추출해야하는 값이 있는 경우 "output" block에 선업합니다. 
이 값은 "terraform apply" 실행 시 사용자에게 보여집니다.
이 모듈을 사용하는 경우 이 값을 변수처럼 사용할 수 있습니다. 

"kubernetes_cluster_name"는 이 output의 식별자이고
value는 생성한 자원의 이름입니다.

```
$ vi outputs.tf
output "kubernetes_cluster_name" {
  value       = google_container_cluster.primary.name
  description = "GKE Cluster Name"
}
```

### 4.버전 정의

지금까지 작성한 Terraform 파일을 실행하기 위해 필요한 버전을 선언합니다.

```
$ vi version.tf
terraform {
  required_version = ">= 0.12"
}
```

## Terraform workspace 초기화하기

지금까지 작성한 Terraform 파일을 기준으로 workspace를 초기화합니다.
```
$ terraform init
...

Terraform has been successfully initialized!
```

초기화하면 어떤 일들이 벌어지는지 살펴보겠습니다.
먼저 디렉토리를 조회해보면 ".terraform" 디렉토리가 생성됐습니다.
```
$ ls -all
...
drwxr-xr-x  4 ...  ... .terraform
```

하위에 "environment" 파일 내용을 확인해보면 default라는 값이 있습니다.
workspace를 초기화하면 기본값으로 "default 값이 생성됩니다. 
```
$ cat .terraform/environment 
default
```

"terraform workspace list" 명령을 실행해 workspace 목록을 조회해보면 
default만 있고 "*" 표시(활성화된 workspace)를 확인할 수 있습니다.
```
$ terraform workspace list
* default
```

추가로 plugins 디렉토리의 "selections.json" 파일 내용을 보면 
"main.tf"의 "provider.version"에 선언한 대로 plugin을 설치한 것을 확인할 수 있습니다.
```
$ cat .terraform/plugins/selections.json 
{
  "registry.terraform.io/hashicorp/google": {
    "hash": "h1:UIugsEPd4efky7G86YVdjkW5wwMIIW/WWeFod7RsbbQ=",
    "version": "3.47.0"
  }
}
```

또한, registry 설정도 보이는데 이건 필요한 모듈을 불러오기 위한 설정으로 보입니다.
기본적으로 [public registry](https://registry.terraform.io/)에서 cloud provider가 제공하는 모듈을 가져오고 필요한 모듈을 만들어 업로드하거나 private registry를 만들 수도 있는거 같습니다.
(추후 모듈화 필요성이 있을때 좀 더 공부해봐야겠네요.)
```
$ ls -all .terraform/plugins/registry.terraform.io/hashicorp/google/3.47.0/darwin_amd64
...
-rwxr-xr-x  1 ...  ... terraform-provider-google_v3.47.0_x5
```

## GKE cluster 프로비저닝하기

이제 드디어 GKE cluster를 생성해보겠습니다.

### 1.환경 변수 설정

"variabes.tf" 파일을 작성할 때 변수들에 값을 재할당 할 수 있다고 했습니다.

아예 "terraform.tfvars" 파일에 아래와 같이 선언할 수도 있습니다.

```
project_id = "my-project-gke"
region     = "asia-northeast3"
```

하지만 민감한 정보가 있는 경우 파일로 관리하다가 실수로 git에 업로드할 위험성이 있으므로 아래와 같이 OS 환경변수에 정의하고 이를 사용하는게 좋다고 생각합니다.
- GOOGLE_APPLICATION_CREDENTIALS: 준비하기에서 생성한 service account json file의 경로를 입력(절대 경로)

```
## "GOOGLE_APPLICATION_CREDENTIALS" is a file's path exported from service account.
export GOOGLE_APPLICATION_CREDENTIALS="REPLACE_ME"
export PROJECT_ID="REPLACE_ME"
export REGION="REPLACE_ME"

## e.g
export GOOGLE_APPLICATION_CREDENTIALS="/User/sjyun/.gcp/gke-service-account.json"
export PROJECT_ID="my-project"
export REGION="asia-northeast3"
```

### 2.검증

자원을 생성하기전 먼저 검증을 할 수 있습니다. 
인프라 자원을 변경하는 것은 매우 조심스러운 작업인데 실수로 지워버린다던가 하는 
위험성을 줄여주는 좋은 기능인거 같습니다.

"terraform plan" 명령을 실행하고 위에서 설정한 환경변수를 Terraform 변수에 할당합니다. "main.tf"에서 정의한대로 vpc network, subnet network, gke cluster, gke node pool이 어떤식으로 생성될지 미리 보여줍니다.

```
$ terraform plan \
    -var "project_id=$PROJECT_ID" \
    -var "region=$REGION" \
    -var "gke_num_nodes=1"
...
Terraform will perform the following actions:
  ...
  # google_compute_network.vpc will be created
  + resource "google_compute_network" "vpc" {
  ...
  # google_container_cluster.primary will be created
  + resource "google_container_cluster" "primary" {
  ...
  # google_container_node_pool.primary_nodes will be created
    + resource "google_container_node_pool" "primary_nodes" {
  ...

Plan: 4 to add, 0 to change, 0 to destroy.
```

### 3.생성

검증을 완료했으니 실제 자원을 생성해보겠습니다.
"terraform apply" 명령을 실행하고 위에서 설정한 환경변수를 Terraform 변수에 할당합니다. "plan" 명령과 다른 점은 실제로 이 명령을 수행할 것인지 한번 더 물어본다는 점입니다.

"yes"를 입력하면 인프라 자원 생성이 시작됩니다.
"Creating..."이라는 메세지와 경과 시간이 계속 출력되고 
단계별로 완료되면 "Creation complete" 메세지가 중간에 출력됩니다.
전체 단계가 완료되면 "Outputs"에 "outputs.tf에 정의한 값이 출력됩니다.

```
$ terraform apply \
    -var "project_id=$PROJECT_ID" \
    -var "region=$REGION" \
    -var "gke_num_nodes=1"
...
Plan: 4 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

google_compute_network.vpc: Creating...
google_compute_network.vpc: Still creating... [10s elapsed]
google_compute_network.vpc: Creation complete after 26s
...
google_compute_subnetwork.subnet: Creation complete after 15s
...

Outputs:
  + kubernetes_cluster_name = "my-project-gke"
  + region                  = "asia-northeast3"

Apply complete! Resources: 4 added, 0 changed, 0 destroyed.
```

## GKE cluster 확인

생성한 Kubenetes cluster에 접속해보겠습니다.
gcloud CLI를 통해 credential 정보를 저장합니다.

```
$ gcloud container clusters get-credentials my-project-gke \
  --region asia-northeast3 \
  --project my-project
Fetching cluster endpoint and auth data.
kubeconfig entry generated for my-project-gke.
```

"node-pools"의 정보를 조회합니다. Machine type이 기본값인 "e3-medium"으로 생성된것을 확인할 수 있습니다.

```
$ gcloud container node-pools list \
  --cluster=$PROJECT_ID-gke \
  --project=$PROJECT_ID \
  --region=$REGION
NAME                         MACHINE_TYPE  DISK_SIZE_GB  NODE_VERSION
my-project-gke-node-pool     e2-medium     100           1.16.15-gke.4300
```

kubectl CLI를 통해 node 정보를 가져옵니다. 3개의 node 정보를 정상적으로 가져오는것을 확인할 수 있습니다.
```
$ kubectl get nodes
NAME                                                  STATUS   ROLES    AGE   VERSION
gke-my-project-gk-my-project-gk-1fbd2278-0vzr   Ready    <none>   10m   v1.16.15-gke.4300
gke-my-project-gk-my-project-gk-7f696cdb-g9f8   Ready    <none>   10m   v1.16.15-gke.4300
gke-my-project-gk-my-project-gk-e52afe54-8cvt   Ready    <none>   10m   v1.16.15-gke.4300
```

## GKE cluster 변경하기

이번에는 GKE cluster의 설정을 변경해보겠습니다.
Machine type을 기본값인 "e2-medium"에서 "e2-standard-2"로 변경해보겠습니다.

적용전에 검증을 합니다. 기존 machine type의 node_pool을 삭제하고 새로운 node_pool이 추가되는 것을 확인할 수 있습니다.
```
$ terraform plan \
    -var "project_id=$PROJECT_ID" \
    -var "region=$REGION" \
    -var "gke_num_nodes=1" \
    -var "gke_machine_type=e2-standard-2"
...
Terraform will perform the following actions:
  # google_container_node_pool.primary_nodes must be replaced
  ~ machine_type      = "e2-medium" -> "e2-standard-2" # forces replacement

Plan: 1 to add, 0 to change, 1 to destroy.
```

실제 적용을 해보겠습니다. 기존 node pool을 삭제하고 새로운 node pool을 생성하는것이 확인됩니다.

<p class="tip-title">참고</p>
<p class="tip-content">
node pool이 삭제되면서 실행중인 pod도 전부 삭제됩니다. 실제 개발/운영 환경이라면 node pool을 여러개 만들어 놓고
변경 작업이 일어나는 node의 pod를 모두 다른 node로 migration하는 작업이 선행되어야 합니다.
Lagacy 환경에서는 어렵고 시간이 많이 소요되는 작업이 Cloud, Kubernetes 환경에서는 좀 더 쉽고 빠르게 가능합니다
</p>

```
$ terraform apply \
    -var "project_id=$PROJECT_ID" \
    -var "region=$REGION" \
    -var "gke_num_nodes=1" \
    -var "gke_machine_type=e2-standard-2"
...
Plan: 1 to add, 0 to change, 1 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

google_container_node_pool.primary_nodes: Destroying... [id=projects/my-project/locations/asia-northeast3/clusters/my-project-gke/nodePools/my-project-gke-node-pool]
...
google_container_node_pool.primary_nodes: Creation complete after 1m22s [id=projects/my-project/locations/asia-northeast3/clusters/my-project-gke/nodePools/my-project-gke-node-pool]
...
Apply complete! Resources: 1 added, 0 changed, 1 destroyed.

Outputs:

kubernetes_cluster_name = my-project-gke
```

변경 작업이 완료되면 node pool을 다시 조회해봅니다. Machine type이 "e2-standard-2"으로 변경되었습니다.
```
$ gcloud container node-pools list \
  --cluster=$PROJECT_ID-gke \
  --project=$PROJECT_ID \
  --region=$REGION
NAME                         MACHINE_TYPE   DISK_SIZE_GB  NODE_VERSION
my-project-gke-node-pool     e2-standard-2  100           1.16.15-gke.4300
```

## GKE cluster 삭제하기

이제 더 이상 사용 계획이 없는 cluster를 삭제하겠습니다.
```
$ terraform destroy \
    -var "project_id=$PROJECT_ID" \
    -var "region=$REGION"
...
Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes
  ...

Destroy complete! Resources: 4 destroyed.
```

## 맺음말

지금까지 Terraform을 사용해 GKE를 생성, 변경 및 삭제하는 방법에 대해 살펴봤습니다.

사실 이 정도의 변경 작업은 GCP 웹콘솔이나 CLI로도 쉽게 할 수 있는 작업입니다.
하지만 인프라 규모가 커지고 협업인원이 늘어날수록 점점 수작업으로 관리하기가 어려워집니다.

방화벽 오픈 규칙을 변경하는 작업을 예로 들어보겠습니다. 
변경작업을 하는 사람이 콘솔이나 CLI를 통해 작업을 한 뒤 혼자만 알고 있다가 퇴사한다고 가정해보겠습니다.
(엑셀이나 시스템에 기록을 해둔다면 그나마 낫겠지만 이마저도 안하는 경우가 있고 다른 사람이 수 많은 내역을 일일히 찾기는 쉽지 않습니다)
이렇게 시스템이 방치된 상태에서 후임자는 특정 규칙이 왜 추가되었는지 파악하기가 쉽지 않습니다. 이는 결국 시스템 장애나 보안사고로도 이어질 수 있습니다.

Terraform과 같은 IaC(Infrastructure as Code) 방식으로 인프라를 관리할 경우 변경 내역을 더 쉽게 파악할 수 있을거이라 생각합니다.
하지만 코드 한줄로 많은 변경작업이 일어날 수 있어 기존 방식보다는 영향도가 클 것이라고 생각합니다. 또한 예상치 못한 상황이 생길 수도 있습니다.
따라서 개발계에서 충분히 사용해보면서 예상치 못한 문제들을 겪어보고 해결한 후에 운영단에 점진적으로 적용하는 편이 좋을 것이라 생각합니다.

개인적으로는 테스트 용도로 인프라 자원의 생성과 삭제를 반복해야하는 경우 엄청 편리하다고 생각합니다.
이렇게 시작해서 점차적으로 사용을 늘려가다보면 자연스럽게 스킬과 경험치가 올라가지 않을까 생각합니다.

이 문서에 없는 더 자세한 내용은 [Kubernetes Engine](https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/container_cluster)을 참고하세요.