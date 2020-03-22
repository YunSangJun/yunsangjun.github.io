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

## Ansible 이란?

[Ansible](https://github.com/ansible/ansible)은 소프트웨어 프로비저닝, 구성 관리를 지원하는 자동화 도구입니다.
"Michael DeHaan"에 의해 개발되었으며 이후 레드햇에 인수되었다고 합니다.

저 같은 경우 이직 후 새로운 개발/운영 환경에서 업무를 하게되면서 Ansible을 처음 접하게 되었습니다.
새로운 개발/운영 환경이란 것이 On-premise(온프레미스) 환경을 말하는데 서버마다 일일히 설정/구성 해줘야하는 작업이 있습니다.

오픈소스 소프트웨어를 물리적인 서버에 구성하기 위해 서버에 계정, 네트워크 등의 설정을 해야하는 경우가 있었는데
한 두대도 아니고 수십~수백대의 서버에 동일한 설정을 반복해야한다고 생각하니 너무 귀찮고 번거로운 일이었습니다.

이렇게 많은 물리 서버의 구성을 해본 경험이 없어서 처음에는 쉘 스크립트로 자동화를 해야겠다고 생각하던 차에
Ansible이라는 자동화 도구를 접하게 되었습니다. 

여러가지 활용 사례를 찾아보니 제가 개발하고자 하는 방향에 적합했고 
쉘 스크립트로 구현해야하는 많은 부분들을 이미 기능으로 제공하고 있어 장점이 많다고 생각합니다.

<p class="tip-title">참고</p>
<p class="tip-content">
이 가이드의 Ansible host(Ansible을 설치하고 있는 서버)에서 사용하는 계정은 sudo 권한이 없는 일반 사용자입니다.
(sudo: 일반 사용자가 root 권한을 일시적으로 획득하여 특정 명령을 실행 할 수 있도록 하는 명령)
</p>

## 1.Python 설치

Ansible을 설치하기 위해서는 먼저 python을 설치해야합니다.

[Python](https://www.python.org/downloads/) 사이트에서 자신의 환경에 맞는 버전을 다운로드 및 설치합니다.

<p class="tip-title">참고</p>
<p class="tip-content">
이 가이드는 Python 3.7 버전을 기준으로 작성했습니다.
</p>

## 2.Ansible 설치하기

이제 Ansible을 설치해보겠습니다.

[Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#latest-releases-via-pip)
사이트의 설치 가이드를 보면 다양한 방법이 있습니다.

이 가이드에서는 OS 환경에 종속성이 없는 
[pip](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#latest-releases-via-pip)
을 통해 설치해보겠습니다.

아래 명령을 실행해 ansible을 설치합니다. "--user" 옵션으로 설치를 권장합니다.

```
//for local
default-user@ansible-host:~$ pip install --user ansible

//for global
default-user@ansible-host:~$ pip install ansible
```

<p class="tip-title">참고</p>
<p class="tip-content">
"--user" 옵션으로 설치한 경우 아래와 같이 환경 변수를 설정해야 합니다.
</p>

```
default-user@ansible-host:~$ PATH=$PATH:$HOME/.local/bin
default-user@ansible-host:~$ export PATH
```

설치가 완료되면 버전을 확인합니다.

```
default-user@ansible-host:~$ ansible --version
default-user@ansible-host:~$ ansible 2.9.2
...
```

## 3.Getting Started

이제 설치를 했으니 간단한 ansible 명령어를 실행해보겠습니다.

Loopback(ansible host)을 호출하고 응답이 오는지 확인하는 명령을 실행합니다.
- host 명: loopback 주소인 "localhost" 입력
- 모듈 명: "-m" 옵션에 "ping" 모듈 입력

"pong" 이라는 응답이 오면 정상적으로 동작한 것입니다.

```
default-user@ansible-host:~$ ansible localhost -m ping
...
localhost | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

위에서 [모듈](https://docs.ansible.com/ansible/latest/user_guide/modules_intro.html)이란 개념이 나왔는데 간략히 살펴보고 지나가겠습니다. 모듈은 일종의 플러그인입니다. Ansible은 모듈을 remote host에서 실행하고 return 값을 수집합니다.

사실 모듈을 사용하지 않고 쉘 스크립트를 사용해서 원하는 명령을 실행할 수도 있습니다.
예를 들어 remote host의 특정 파일을 복사하는 명령을 실행한다고 가정해보겟습니다.

아래 명령은 쉘 스크립트를 사용해 remote host의 파일을 복사하는 명령을 실행합니다. 
```
default-user@ansible-host:~$ ansible all -m shell -a "cp ~/.profile ~/.profile.bak"
```

같은 명령을 "copy" 모듈을 사용할 수도 있습니다.
```
default-user@ansible-host:~$ ansible all -m copy -a "src=~/.profile dest=~/.profile.bak"
```

모듈을 사용하면 몇 가지 장점이 있습니다.
- 멱등성을 보장합니다. 즉 실행 결과를 저장해 다시 실행 시 변경된 부분만 실행할 수 있습니다.
- OS에 대한 호환성이 있습니다. 다른 OS에서 실행할 때 명령어를 다시 작성할 필요가 없습니다.

어떤 방법을 사용하는지는 개인의 선택이지만 유지보수 측면을 고려한다면 모듈을 사용하는 것이 효율적입니다.

## 4.Host 파일 구성하기

Ansible host에서 여러대의 remote host를 호출해야 할 경우 "/etc/ansible/hosts" 파일에 등록해서 사용합니다.

"/etc/ansible/hosts" 파일을 생성하고 remote host를 입력합니다.
"[example]"는 remote host들의 그룹 이름입니다. 딱히 정해진 이름은 없으니 자유롭게 정하면됩니다. 

```
default-user@ansible-host:~$ vi /etc/ansible/hosts
[example]
host01.example.com
host02.example.com
host03.example.com
```

<p class="tip-title">참고</p>
<p class="tip-content">
Ansible host에서 remote host로 ssh 클라이언트를 통해 접속 가능한 상태이어야합니다.
일반적으로 ssh key를 통해 접속합니다.
</p>

## 5. Remote host에 ping 명령 호출하기

이제 등록한 remote host에 ping 명령을 호출해보겠습니다.
- host 명: 그룹명 "example" 입력
- 모듈 명: "-m" 옵션에 "ping" 모듈 입력

```
default-user@ansible-host:~$ ansible example -m ping
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
default-user@ansible-host:~$ vi /etc/ansible/hosts
[example01]
host01.example.com

[example02]
host02.example.com

[example03]
host03.example.com

default-user@ansible-host:~$ ansible all -m ping
...
```

## 맺음말

지금까지 Ansible이 무엇인지 알아보고 간단한 사용 방법에 대해 알아봤습니다.

서버를 구성 할 때 필수적으로 하는 작업(그리고 가장 번거로운)이 있습니다. 서버의 계정을 생성하는 작업인데 이 작업을 
[Ansible을 활용해 자동화](/automation/2019/10/15/ansible-create-account.html)하는 방법에 대해 알아보겠습니다.

