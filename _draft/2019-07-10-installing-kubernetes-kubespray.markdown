---
layout: post
title:  "Kubespray를 활용하여 Kubernetes 설치하기"
author: 윤상준
date: 2019-07-10
categories: kubernetes
tags:
- kubernetes
- kubespray
---

이 문서에서는 Kubespray를 활용하여 Kubernetes를 설치해보겠습니다.

## 사전 준비

먼저 Kubernetes를 설치하기 위한 인프라 환경 구성을 해보겠습니다.

Kubespray는 설치 환경으로 GCP, Azure, AWS, OpenStack, vSphere와 같은 Cloud, 그리고 Baremetal 환경을 지원합니다.

이 문서에서는 AWS를 활용해보겠습니다.

### Terraform을 활용하여 설치에 필요한 AWS 환경 구성하기

- Terraform 클라이언트 툴 0.8.7 이상 버전을 설치합니다.

    <p class="warning-title">경고</p>
    <p class="warning-content">
    terraform 버전은 0.11.x 하위 버전을 사용할 것을 권장합니다.
    0.12.x 버전부터 API 변경으로 설정 파일 수정이 필요합니다.
    이 문서에서는 0.11.14 버전을 기준으로 작성했습니다.
    </p>

- [Kubespray Github 프로젝트](https://github.com/kubernetes-sigs/kubespray)를 다운로드 받습니다. 
```
git clone https://github.com/kubernetes-sigs/kubespray.git
cd kubespray/contrib/terraform/aws
```

- AWS EC2 콘솔 접속 > 네트워크 및 보안 > 키 페어 > 키 페어 생성

- 로컬 환경의 터미널에서 AWS 환경 변수를 export합니다.
```
export TF_VAR_AWS_ACCESS_KEY_ID="xxx"
export TF_VAR_AWS_SECRET_ACCESS_KEY="xxx"
export TF_VAR_AWS_SSH_KEY_NAME="위에서 생성한 키 페어 이름"
export TF_VAR_AWS_DEFAULT_REGION="xxx"
```

- terraform.tfvars 파일의 내용을 원하는 구성으로 변경합니다.
변경할 사항이 없으면 기본 설정을 사용합니다.

- 아래 명령을 실행해 terraform 모듈을 다운로드합니다. 
```
$ terraform init
Initializing modules...
...
Terraform has been successfully initialized!
```

- 아래 명령을 실행해 Kubernetes 설치에 필요한 자원을 생성합니다.
```
$ terraform apply
Apply complete! Resources: 47 added, 0 changed, 0 destroyed.
Outputs:
...
```