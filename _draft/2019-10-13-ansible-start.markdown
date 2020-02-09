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
- devops
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

## 1.Python 설치

Ansible을 설치하기 위해서는 먼저 python을 설치해야합니다.

[Python](https://www.python.org/downloads/) 사이트에서 자신의 환경에 맞는 버전을 다운로드 및 설치합니다.

<p class="tip-title">참고</p>
<p class="tip-content">
이 가이드는 Python 3.x 버전을 기준으로 작성했습니다.
</p>

## 2.Ansible 설치하기

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

## 3.Getting Started

이제 설치를 했으니 간단한 ansible 명령어를 실행해보겠습니다.

Loopback(ansible host)을 호출하고 응답이 오는지 확인하는 명령을 실행합니다.
- 1번째 옵션: 호출할 host 명(127.0.0.1)
- 2번째 옵션: "-m" 옵션에 실행할 모듈명을 입력(ping)

"pong" 이라는 응답이 오면 정상적으로 동작한 것입니다.

```
$ ansible 127.0.0.1 -m ping
...
127.0.0.1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

## 4.Host 파일 구성하기

Ansible host에서 여러대의 remote host를 호출해야 할 경우 `/etc/ansible/hosts` 파일에 등록해서 사용합니다.

"/etc/ansible/hosts" 파일을 생성하고 remote host를 입력합니다.
"[hosts]"는 remote host들의 그룹 이름입니다. 딱히 정해진 이름은 없으니 자유롭게 정하면 됩니다. 

```
$ vi /etc/ansible/hosts
[hosts]
host01.example.com
host02.example.com
host03.example.com
```

<p class="tip-title">참고</p>
<p class="tip-content">
Ansible host에서 remote host로 ssh 클라이언트를 통해 접속 가능한 상태이어야합니다.
</p>

## 5. Remote host에 ping 명령 호출하기

이제 등록한 remote host에 ping 명령을 호출해보겠습니다.
- 1번째 옵션: 그룹명(hosts) 입력
- 2번째 옵션: "-m" 옵션에 실행할 모듈명을 입력(ping)

```
$ ansible hosts -m ping
host01.example.com | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
host02.example.com | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
host03.example.com | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
```

그룹명 대신 "all"을 입력해도 동일한 결과가 출력됩니다.
"/etc/ansible/hosts" 파일에 여러개의 host 그룹을 등록할 수 있는데
전체 그룹에 대해 호출하고 싶은 경우 "all"을 사용하면됩니다.

```
$ ansible all -m ping
...
```

## 5.Remote host의 hostname 조회

이번에는 remote host의 hostname을 조회해보겠습니다.
- 1번째 옵션: 그룹명(hosts) 또는 "all" 입력
- 2번째 옵션: "-m" 옵션에 실행할 모듈명을 입력(shell)
- 3번째 옵션: "-a" 옵션에 remote host에서 실행할 shell 명령어 입력

```
$ ansible all -m shell -a "hostname"
host01.example.com | CHANGED | rc=0 >>
host01.example.com
host02.example.com | CHANGED | rc=0 >>
host02.example.com
host03.example.com | CHANGED | rc=0 >>
host03.example.com
```