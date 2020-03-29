---
layout: post
title:  "Ansible Playbook 시작하기"
author: sj
date: 2019-11-07
categories: automation
tags:
- ansible
- ansibleplaybook
- automation
- devops
---

## Overview

Ansible을 활용하면 반복적인 시스템 작업들을 자동화 할 수 있습니다.
하지만 명령어 기반 작업이다 보니 엔지니어마다 작성하는 방식이 다르고 표준을 정하기도 어렵습니다.

게다가 하나의 명령을 수행하는 것이 아니라 여러가지 명령을 조합해서 한 번에 실행해야하는 경우는 어떨까요?
물론 쉘 스크립트로 명령을 조합해서 원하는 결과는 얻을 수 있을 것입니다.
하지만 분명 코드는 더 복잡해지고 형상 관리가 어려워질 것입니다.

저 또한 동일한 Ansible 명령을 반복적으로 수행하다보니 이것 또한 번거롭다고 느껴지고 형상관리를 해야 할 필요성을 느꼈습니다. 
또 자동화 코드를 선언적인 형태로 작성해 누구나 알아보기 쉽고 유지보수하기 쉬웠으면 좋겠다고 생각했습니다.

해서 여러가지 방법을 찾아보던 중 [Ansible Playbook](https://docs.ansible.com/ansible/latest/cli/ansible-playbook.html)에 대해 알게되었고 
제가 원하는 방향과 맞는 거 같아 자동화 업무에 사용하고 있습니다.

Ansible Playbook을 처음 접하는 분이시라면 Ansible과 차이점에 대해 궁금하실텐데요.
간단하게 설명해 Ansible이 작업장의 도구라면 Ansible Playbook은 작업 메뉴얼이고 Inventory는 재료라고 비유할 수 있습니다.

Ansible Playbook에 대한 자세한 내용은 간단한 예제를 통해 살펴보겠습니다.

## Getting Started

[Ansible을 활용해 서버 계정 생성 자동화](/automation/2019/10/15/ansible-create-account.html) 방법에 대해 설명한 적이 있었는데요.
이 작업을 Ansble playbook을 사용해 자동화하는 방법에 대해 알아보겠습니다.

서버 계정 생성 자동화 작업은 아래의 단계로 수행됩니다.
Ansible을 사용해서 각 단계의 작업을 수행하는 명령을 실행했는데요.
이를 Ansible Playbook으로 변경해보겠습니다.

1. 사용자 계정 추가하기
2. 패스워드 설정하기
3. sudo 명령어 사용 권한 추가하기

<p class="tip-title">참고</p>
<p class="tip-content">
이 문서의 Ansible host(Ansible을 설치하고 있는 서버)에서 사용하는 계정은 sudo 권한이 없는 일반 사용자입니다.
(sudo: 일반 사용자가 root 권한을 일시적으로 획득하여 특정 명령을 실행 할 수 있도록 하는 명령)
</p>

### 1. Playbook 생성

"create-account.yaml" 이라는 파일을 생성합니다.
그리고 아래와 같이 작성해보겠습니다.

```yaml
---
{% raw %}
- name: Create account
  hosts: "{{ HOST_NAME }}"
  become: true
  tasks:
    - name: create user
      user:
        name: "{{ USER_NAME }}"
    - name: set password
      user:
        name: "{{ USER_NAME }}"
        password: "{{ PASSWORD | password_hash('sha512') }}"
    - name: create sudoers
      copy:
       content: |
         {{USER_NAME}} ALL=(ALL) NOPASSWD:ALL
       dest: "/etc/sudoers.d/{{USER_NAME}}"
       owner: root
       group: root
       mode: 0664
{% endraw %}
```

Playbook은 yaml 포맷으로 선언적으로 작성합니다. 파일 위에서 부터 차례로 살펴보겠습니다.

- name: Playbook의 이름
- hosts: Remote host 이름
- become: 명령을 실행하는 사용자 계정의 권한을 승격
- tasks: Remote host에서 수행할 작업들

{% raw %}"{{ HOST_NAME }}"{% endraw %}과 같이 선언한 부분은 playbook 실행 시 입력받을 인자 값입니다.

다음으로 수행하는 각 작업(tasks)에 대해 살펴보겠습니다.

- "create user" task
    - user: user 모듈 사용 선언
        - name: 생성할 사용자 계정 이름

- "set password" task
    - user: user 모듈 사용 선언
        - name: 생성할 사용자 계정 이름
        - password: 생성할 사용자 계정 패스워드

- "create sudoers" task
    - copy: copy 모듈 사용 선언
        - content: 복사할 문자열
        - dest: 복사할 목적지로 remote host의 절대 경로
        - mode: 복사할 목적지의 파일/디렉토리 권한 입력
        - owner: 복사할 목적지의 파일/디렉토리 소유자
        - group: 복사할 목적지의 파일/디렉토리 그룹

코드를 선언적으로 작성했기 때문에 누가봐도 어떤 작업을 수행하는지 알기 쉽고 
작업 순서 또한 명확히 파악됩니다. 

### 2. Playbook 실행

이제 생성한 "create-account.yaml" playbook을 실행해보겠습니다.
Python Ansible 패키지를 설치하면 함께 설치되는 "ansible-playbook" 명령을 실행합니다.

- "i" 옵션: Remote host의 정보를 가지고 있는 inventory 파일 경로를 지정
- "--extra-vars" 옵션: Playbook에 전달할 인자 값
  - HOST_NAME: Remote host 이름
  - USER_NAME: 생성할 사용자 계정 이름
  - PASSWORD: 생성할 사용자 계정 패스워드

```bash
default-user@ansible-host:~$ ansible-playbook -i [path/to/inventory] create-account.yaml \
--extra-vars "HOST_NAME=[HOST_NAME] USER_NAME=[USER_NAME] PASSWORD=[PASSWORD]"
```

아래와 같이 "test" 사용자 계정을 모든 host에 생성해보겠습니다.

```bash
default-user@ansible-host:~$ ansible-playbook -i /etc/ansible/hosts create-account.yaml \
--extra-vars "HOST_NAME=all USER_NAME=test PASSWORD=test1234"

PLAY [Create account] *******************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************

TASK [create user] **********************************************************************************************************************************
- name: Create account
changed: [host03.example.com]
changed: [host02.example.com]
changed: [host01.example.com]

TASK [set password] *********************************************************************************************************************************
changed: [host02.example.com]
changed: [host03.example.com]
changed: [host01.example.com]

TASK [create sudoers] *******************************************************************************************************************************
changed: [host03.example.com]
changed: [host02.example.com]
changed: [host01.example.com]

PLAY RECAP ****************************************************************************************************************************************** 
host01.example.com         : ok=4    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
host02.example.com         : ok=4    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
host03.example.com         : ok=4    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```
실행 결과를 보면 Playbook에 정의한 각 task 들이 순차적으로 실행되는 것을 확인할 수 있습니다.
마지막에 "PLAY RECAP"에 "changed"의 숫자는 작업에 성공해 실제 반영되 task 수를 나타냅니다.

Playbook에 3가지의 task를 정의했기 때문에 3이 표시되는 것으로 볼 수 있습니다.
만약 암호만 변경해서 다시 실행한다면 실제 반영된 task 수는 몇으로 표시될까요? 
3가지 task 중 암호만 변경했으므로 1로 표시될 것입니다.

계정을 생성했으니 실제 로그인이 되는지 확인해보겠습니다.

```bash
## Remote host에 접속
default-user@ansible-host:~$ ssh host01.example.com

## test 사용자 계정으로 로그인
default-user@host01.example.com:~$ su - test
Password: ...

## sudo 명령으로 root 계정으로 로그인
test@host01.example.com:~$ sudo su - root

## root 계정으로 정상 로그인됨
root@host01.example.com:~#
```
<p class="tip-title">참고</p>
<p class="tip-content">
암호를 입력했으므로 명령어를 실행한 뒤 history를 clear 해주는 것이 좋습니다.
</p>

## 맺음말

지금까지 Ansible Playbook에 대해 알아봤습니다.

처음에 Ansible은 도구, Inventory는 재료이고 Playbook은 작업 메뉴얼이라고 했는데요.
이 문서를 읽고 이 뜻이 충분히 이해가 됐을지 모르겠습니다.

여기까지 충분히 이해가 되었다면
다음으로 [Ansible Playbook 응용](/automation/2019/11/18/ansible-playbook-advanced.html)
단계에서 좀 더 다양한 기능을 살펴보세요.