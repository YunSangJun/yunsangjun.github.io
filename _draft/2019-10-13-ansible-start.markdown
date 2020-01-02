---
layout: post
title:  "Ansible 시작하기"
author: sj
date: 2019-10-13
categories: automation
tags:
- ansible
- ansibleplaybook
- automation
- ops
---

## Overview

[Ansible](https://github.com/ansible/ansible)은 소프트웨어 프로비저닝, 구성 관리를 지원하는 자동화 도구입니다.
"Michael DeHaan"에 의해 개발되었으며 이후 레드햇에 인수되었다고 합니다.

저 같은 경우 이직 후 새로운 개발/운영 환경에서 업무를 하게되면서 Ansible을 처음 접하게 되었습니다.
오픈소스 소프트웨어를 베어메탈 서버에 구성하기 위해 서버에 계정, 네트워크 등의 설정을 해야하는 경우가 있었는데
한 두대도 아니고 수 백대의 서버에 설정을 하려니 너무 귀찮고 번거로운 일이었습니다.

이렇게 많은 서버의 구성을 해본 경험이 없어서 처음에는 쉘 스크립트로 자동화를 해야겠다고 생각하던 차에
Ansible이라는 자동화 도구를 접하게 되었습니다. 

여러가지 활용 사례를 찾아보니 제가 개발하고자 하는 방향에 적합했고 
쉘 스크립트로 구현해야하는 많은 부분들을 이미 기능으로 제공하고 있어 장점이 많다고 생각합니다.

## 1.1.Python 설치

Ansible을 설치하기 위해서는 먼저 python을 설치해야합니다.

[Python](https://www.python.org/downloads/) 사이트에서 자신의 환경에 맞는 버전을 다운로드 및 설치합니다.

<p class="tip-title">참고</p>
<p class="tip-content">
이 가이드는 Python 3.x 버전을 기준으로 작성했습니다.
</p>

## 1.Ansible 설치하기

이제 Ansible을 설치해보겠습니다.

[Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#latest-releases-via-pip)
사이트의 설치 가이드를 보면 다양한 방법이 있습니다.

이 가이드에서는 OS 환경에 종속성이 없는 
[pip](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#latest-releases-via-pip)
을 통해 설치해보겠습니다.

아래 명령을 실행해 ansible을 설치합니다. `--user` 옵션으로 설치를 권장합니다.

```
//for local
pip install --user ansible

//for global
pip install ansible
```

<p class="tip-title">참고</p>
<p class="tip-content">
"--user" 옵션으로 설치한 경우 아래와 같이 환경 변수를 설정해야 합니다.
</p>

```
PATH=$PATH:$HOME/.local/bin
export PATH
```

설치가 완료되면 버전을 확인합니다.

```
# ansible --version
ansible 2.9.2
...
```

## 2.Host 파일 구성하기

## 3.Getting Started

이제 Ansible을 사용하기 위한 준비를 마쳤습니다.

간단하게 Ansible 명령을 통해 host 파일에 등록된 서버들에서 hostname을 가져와보겠습니다.

## 4.서버에 계정 구성하기


