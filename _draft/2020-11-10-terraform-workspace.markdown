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
- workspace
---

Terraform으로 인프라 자원을 생성하면 terraform state(terraform.tfstate.d) 파일이 생성됩니다.
그리고 이 파일에는 생성한 자원의 상태 정보가 기록됩니다.

Terrform을 사용해 인프라 자원을 관리하다가 문득 이런 의문점이 생겼습니다.
만약 개발/운영 환경과 같이 일부 속성만 다른 경우 terraform 파일을 두 버전으로 관리해야하나?

예를 들어, Terraform을 사용해 개발/운영계로 사용할 GKE cluster를 각각 생성한다고 가정해보겠습니다. 

## 준비하기

[Terraform으로 GKE 관리하기](/automation/2020/11/09/terraform-gke.html)에서 사용한 
[Terraform 파일 저장소](https://github.com/YunSangJun/terraform-resources.git)를 다운로드하세요.

다음 과정에서 이 파일을 사용하겠습니다.

## 개발계 workspace 생성하기

1. workspace 목록 조회

    위에서 다운로드한 저장소 디렉토리로 이동한 뒤 Terrafrom 초기화를 합니다.
    ```
    $ cd terraform-resources
    $ terraform init
    ```

    먼저 현재 workspace 목록을 조회해보겠습니다.
    "terraform init"을 통해 초기화를 한 경우 "default" workspace만 존재합니다.
    ```
    $ terraform workspace list
    * default
    ```

2. workspace 생성

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
    -rw-r--r--  1 ...  110524863   851 11 10 00:58 terraform.tfstate
    drwxr-xr-x  4 ...  110524863   128 11 10 01:14 terraform.tfstate.d

    $ ls -all terraform.tfstate.d
    ...
    drwxr-xr-x  4 ...  110524863  128 11 10 01:11 my-dev
    ```

3. 자원 생성

    이제 개발계 용도로 사용할 GKE cluster를 생성해보겠습니다.
    ```
    export GOOGLE_APPLICATION_CREDENTIALS="~/.credential/gke-service-account-dev.json"
    export PROJECT_ID="my-dev-295114"
    export REGION="asia-northeast3"

    $ terraform apply \
        -var "project_id=$PROJECT_ID" \
        -var "region=$REGION" \
        -var "gke_num_nodes=1"
    ...

    Outputs:
    kubernetes_cluster_name = my-dev-295114-gke
    region = asia-northeast3
    ```

4. terraform state 확인

    이번에는 생성한 자원에 대한 terraform state를 확인해보겠습니다.
    "my-dev" 디렉토리 하위에 "terraform.tfstate" 파일이 생성된 것이 보입니다.
    ```
    $ ls -all terraform.tfstate.d/my-dev
    -rw-r--r--  1 ...  110524863  12092 11 10 01:11 terraform.tfstate
    ```

    이 파일 내용을 확인해보면 개발계 GKE cluster 정보가 기록되어 있는 것을 확인할 수 있습니다.
    ```
    $ cat terraform.tfstate.d/my-dev/terraform.tfstate |more
    {
    ...
    "outputs": {
        "kubernetes_cluster_name": {
        "value": "my-dev-295114-gke",
        "type": "string"
        },
    ...
    ```

## 운영계 workspace 생성하기

1. workspace 생성

    운영계 용도의 "my-prod" workspace를 생성하겠습니다.
    ```
    $ terraform workspace new my-dev
    Created and switched to workspace "my-prod"!

    $ terraform workspace list
    default
    my-dev
    * my-prod
    ```

2. 자원 생성

    이제 운영계 용도로 사용할 GKE cluster를 생성해보겠습니다.
    운영계는 machine type을 다르게 설정했습니다.
    ```
    export GOOGLE_APPLICATION_CREDENTIALS="~/.credential/gke-service-account-prod.json"
    export PROJECT_ID="my-prod-295114"
    export REGION="asia-northeast3"

    $ terraform apply \
        -var "project_id=$PROJECT_ID" \
        -var "region=$REGION" \
        -var "gke_num_nodes=1" \
        -var "gke_machine_type=e2-standard-2"
    ...

    Outputs:
    kubernetes_cluster_name = my-prod-295114-gke
    region = asia-northeast3
    ```

3. terraform state 확인

    "my-prod" 디렉토리 하위에 "terraform.tfstate" 파일이 생성되었고 이 파일 내용을 확인해보면 운영계 GKE cluster 정보가 기록되어 있는 것을 확인할 수 있습니다.
    ```
    $ cat terraform.tfstate.d/my-prod/terraform.tfstate |more
    {
    ...
    "outputs": {
        "kubernetes_cluster_name": {
        "value": "my-prod-295114-gke",
        "type": "string"
        },
    ...
    ```

## 자원 정리

