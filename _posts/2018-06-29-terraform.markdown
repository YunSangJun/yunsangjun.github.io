---
layout: post
title:  "Terraform 사용하기"
author: 윤상준
date: 2018-06-29
categories: terraform
---

Terraform은 인프라를 코드로 관리할 수 있도록 지원하는 오픈소스 프로젝트입니다.

멀티 클라우드를 사용하는 것이 최근 트렌드입니다. 그만큼 다양하고 복잡한 클라우드 자원을 효율적으로 관리하는 것이 중요합니다.

Terraform은 AWS, GCP, Microsoft Azure, OpenStack 등의 멀티 클라우드를 지원하므로 이를 활용해 코드로 인프라 자원을 효율적으로 관리할 수 있습니다.

참고:[Terraform 지원 provider](https://www.terraform.io/docs/providers/index.html)

이 페이지에서는 terraform을 사용하여 IBM Cloud에 가상머신을 생성, 삭제하는 방법에 대해 설명합니다.

## 준비하기

아래 가이드를 참고하여 Terraform 및 softlayer 플러그인을 설치합니다.

이 페이지에서는 아래와 같은 버전을 기준으로 작성했습니다.

- Terraform : v0.11.7
- softlayer 플러그인 : 1.5.0

### Terraform 설치

- 아래 페이지에서 OS별 바이너리 다운로드

    https://www.terraform.io/downloads.html

- OSX

    ```
    mv terraform /usr/local/bin
    ```

### softlayer 플러그인 다운로드

- 아래 페이지에서 OS별 바이너리 다운로드

    https://github.com/softlayer/terraform-provider-softlayer/releases

- 아래와 같이 플러그인 경로 설정

    ```
    # vi ~/.terraformrc
    providers {
        softlayer = "/path/to/bin/terraform-provider-softlayer"
    }
    ```

### Softlayer 포탈에서 ssh key 생성

Terraform을 이용하여 VM을 생성할 때 사용할 ssh key를 설정하는 방법입니다.

- 디바이스 > 관리 > SSH 키 선택 > 추가 탭 클릭

- 퍼블릭 키 및 라벨 입력 후 추가 버튼 클릭(이 페이지에서는 ssh key 이름을 sl_key로 가정)

- Terraform `main.tf`에 위 키를 정의해서 VM 생성시 활용합니다.

    ```
    data "softlayer_ssh_key" "public_key" {
        label = "sl_key"
    }
    ```

## 자원 정의

디렉토리를 생성하고 생성할 자원에 대한 정의를 합니다.

- 자원 정의

    계정, ssh key, VM 등의 자원에 대해 정의합니다.

    자주 사용하는 값들은 변수화 해서 `vars.tf`에 정의합니다.

    ```
    $ vi main.tf

    //계정 정보
    provider "softlayer" {
      username = "${var.username}"
      api_key  = "${var.api_key}"
    }

    //VM에서 사용할 ssh key
    data "softlayer_ssh_key" "public_key" {
        label = "sl_key"
    }

    //VM 정의
    resource "softlayer_virtual_guest" "sample_vm" {
      count                = "${var.sample_vm_count}"                           //VM 개수
      os_reference_code    = "${var.os}"                                        //OS 타입
      public_vlan_id       = "${var.public_vlan_id}"                            //Public VLAN ID
      private_vlan_id      = "${var.private_vlan_id}"                           //Private VLAN ID
      domain               = "${var.domain}"                                    //VM의 도메인 명
      dedicated_acct_host_only = "${var.dedicated_host}"                        //Dedicated VM 생성 여부
      hostname             = "${var.sample_vm_hostname}-0${count.index+1}"      //VM의 호스트 명
      ssh_key_ids          = ["${data.softlayer_ssh_key.public_key.id}"]        //VM에서 사용할 ssh key
      datacenter           = "${var.datacenter}"                                //데이터센터 명
      disks                = [100,250]                                          //디스크 크기. SAN 타입[Primary Disk, Second Disk, ...]
      local_disk           = "${var.local_disk}"                                //로컬 디스크 사용여부
      hourly_billing       = "${var.hourly_billing}"                            //시간 단위 과금 사용여부
      private_network_only = "${var.private_network_only}"                      //VM 생성 시 사설 네트워크만 사용할지 여부. true일 경우 public_vlan_id 주석 처리 필요
      network_speed        = 1000                                               //VM 네트워크 속도. Shared VM 타입은 100Mbps가 최대
      cores                = 2                                                  //VM CPU Core 수
      memory               = 4096                                               //VM Memory 수
      notes                = "${var.notes}"                                     //VM에 대한 태크 명
    }
    ```

- 환경 변수 정의

    `main.tf`에서 사용하는 환경 변수를 정의합니다.

    ```
    $ vi vars.tf

    variable username {}        //환경변수로 입력받기 위해 템플릿만 선언

    variable api_key {}         //환경변수로 입력받기 위해 템플릿만 선언

    variable datacenter {
        default = "seo01"
    }

    variable domain {
        default = "example.com"
    }

    variable dedicated_host {
        default = "false"
    }

    variable os {
        default = "CENTOS_7_64"
    }

    variable hourly_billing {
        default = "true"
    }

    variable private_network_only {
        default = "false"
    }

    variable local_disk {
        default = "false"
    }

    variable public_vlan_id {
        default = xxx
    }

    variable private_vlan_id {
        default = xxx
    }

    variable sample_vm_hostname {
        default = "sample"
    }

    variable sample_vm_count {
        default = 1
    }
    ```

## IBM Account 정보 export

```
export IBM_ACCOUNT=xxx
export API_KEY=xxx
```

## 초기화

아래 명령 실행해 초기화

```
terraform init
```

## 자원 생성

아래 명령 실행해 자원 생성

```
terraform apply -var "username=$IBM_ACCOUNT" -var "api_key=$API_KEY"
```

## 자원 삭제

아래 명령 실행해 자원 삭제

```
terraform destroy -var "username=$IBM_ACCOUNT" -var "api_key=$API_KEY"
```
