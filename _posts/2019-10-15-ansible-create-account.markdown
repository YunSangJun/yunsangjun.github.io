---
layout: post
title:  "Ansible 활용 #1 서버 계정 생성 자동화"
author: sj
date: 2019-10-15
categories: automation
tags:
- ansible
- ansibleplaybook
- automation
- devops
---

## Overview

서버에 무언가 필요한 것들을 설치하기 전에 가장 먼저 하는 일이 사용자 계정을 생성하고 권한을 할당하는 일입니다.

개인적인 스터디로 사용하는 서버라면 root 계정을 사용해도 무방하겠지만 
운영용 서버라면 root 계정은 서버 관리자만 알고 일반 사용자는 계정과 필요한 권한만 할당 받는 것이 일반적입니다.

Ansible의 처음 접하는 분이라면 
[Ansible 시작하기](/blog/automation/2019/10/13/ansible-start.html)에서 
기본적인 개념과 사용 방법에 대해 살펴보는것이 좋습니다.

<p class="tip-title">참고</p>
<p class="tip-content">
이 가이드의 Ansible host(Ansible을 설치하고 있는 서버)에서 사용하는 계정은 sudo 권한이 없는 일반 사용자입니다.
(sudo: 일반 사용자가 root 권한을 일시적으로 획득하여 특정 명령을 실행 할 수 있도록 하는 명령)
</p>

## 1. 계정 추가하기

Remote host에 계정을 추가하는 명령을 실행해보겠습니다.
- host 명: 전체 host 그룹인 "all" 입력
- 모듈 명: "-m" 옵션에 "user" 모듈 입력
- 모듈 파라미터: user 모듈의 name 파라미터에 "example-user" 입력
- become: 명령을 실행하는 사용자 계정의 권한을 승격. "--become" 또는 "-b" 입력

```
default-user@ansible-host:~$ ansible all -m user -a "name=example-user" -b
host01.example.com | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": true,
    "comment": "",
    "create_home": true,
    "group": 1003,
    "home": "/home/example-user",
    "name": "example-user",
    "shell": "",
    "state": "present",
    "system": false,
    "uid": 1002
}
...
```

이 명령을 실행하면 [user 모듈](https://docs.ansible.com/ansible/latest/modules/user_module.html)을 사용해서 "example-user"라는 이름의 계정을 생성합니다.

<p class="tip-title">참고</p>
<p class="tip-content">만약 "-b" 옵션을 추가하지 않는다면 "useradd" 명령을 실행할 권한이 없다는 에러가 발생할 것입니다.</p>
<p class="tip-content">이 에러가 발생하는 이유는 ansible을 실행하는 사용자 계정에 sudo 권한이 없기 때문입니다.
(위에서 언급한 것과 같이 제가 테스트하는 계정도 sudo 권한이 없습니다. 만약 root 계정으로 ansible을 실행한다면 정상적으로 동작할 것입니다.)</p>
<p class="tip-content">하지만 일반적으로 root 계정을 사용하지 않습니다. 따라서 "-b" 옵션을 추가해 명령 실행해 필요한 권한을 할당 받아 사용하는 것입니다.</p>

이번에는 Remote host에 실제로 계정이 생성되었는지 확인해보겠습니다.

- host 명: 전체 host 그룹인 "all" 입력
- 모듈 명: "-m" 옵션에 "shell" 모듈 입력
- 모듈 파라미터: shell 모듈의 파라미터에 "cat /etc/passwd" 명령 입력
- 출력 옵션: "-o" 옵션을 추가해 결과를 한줄로 출력

```
default-user@ansible-host:~$ ansible all -m shell -a "cat /etc/passwd |grep 'example-user'" -o
host01.example.com | CHANGED | rc=0 | (stdout) example-user:x:1002:1003::/home/example-user:
host02.example.com | CHANGED | rc=0 | (stdout) example-user:x:1002:1003::/home/example-user:
host03.example.com | CHANGED | rc=0 | (stdout) example-user:x:1002:1003::/home/example-user:
```

이 명령을 실행하면 remote host의 계정 정보를 가져옵니다.
결과를 확인해보면 "example-user"가 추가된 것을 확인할 수 있습니다.

## 2. 패스워드 추가하기

위에서 생성한 계정에 패스워드를 추가하는 명령을 실행해보겠습니다.
- host 명: 전체 host 그룹인 "all" 입력
- 모듈 명: "-m" 옵션에 "user" 모듈 입력
- 모듈 파라미터
    - name: 위에서 생성한 계정인 "example-user" 입력
    - update_password: "always" 입력(입력한 암호가 기존 패스워드와 다른 경우 업데이트)
    - password: 암호 "example-password"를 hash한 값을 입력

```
default-user@ansible-host:~$ ansible all -m user -a "name=example-user update_password=always password={{ 'example-password' | password_hash('sha512') }}"
```

이 명령을 실행하면 "example-user"라는 이름의 계정에 "example-password" 암호가 설정됩니다.

암호가 정상적으로 설정되었는지 확인하기 위해 remote host에서 신규 생성한 계정으로 로그인을 해보겠습니다.
패스워드 입력 프롬포트가 나타나면 위에서 설정한 패스워드 "example-password"를 입력합니다.
```
default-user@ansible-host:~$ su - example-user 
Password:
```

"example-user" 계정으로 정상 로그인되었습니다.
로그인을 확인했으니 다음 과정을 위해 로그아웃하고 원래 계정으로 돌아가겠습니다.
```
example-user@ansible-host:~$ exit

default-user@ansible-host:~$ 
```

## 3. sudo 명령어 사용 권한 추가하기

마지막으로 신규 생성한 계정에 sudo 명령어를 사용할 수 있는 권한을 추가해보겠습니다.

계정에 권한을 추가하는 방법은 1)"/etc/sudoers" 파일을 직접 수정하는 방법과 
2)"/etc/sudoers.d/" 디렉토리에 파일을 추가하는 방법이 있습니다.

어느 방법을 선택해도 무관하지만 좀 더 안전한 방법인 2번째 방법으로 진행해보겠습니다.

- host 명: 전체 host 그룹인 "all" 입력
- 모듈 명: "-m" 옵션에 복사 기능을 수행하는 "copy" 모듈 입력
- 모듈 파라미터
    - content: 복사할 문자열 입력. "example-user ALL=(ALL) NOPASSWD:ALL"
    - dest: 복사할 목적지로 remote host의 절대 경로 입력. "/etc/sudoers.d/example-user"
    - mode: 복사할 목적지의 파일/디렉토리 권한 입력. "0664"
- become: 명령을 실행하는 사용자 계정의 권한을 승격. "--become" 또는 "-b" 입력

```
default-user@ansible-host:~$ ansible all -m copy -a "content='example-user ALL=(ALL) NOPASSWD:ALL' dest=/etc/sudoers.d/example-user mode=0644" -b
host01.example.com | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": true,
    "checksum": "211b02ac91fd0734b04f766226aaa7bc8fb0aec8",
    "dest": "/etc/sudoers.d/example-user",
    "gid": 0,
    "group": "root",
    "md5sum": "c10df13e1e8647a0a77e2a8343678fe5",
    "mode": "0644",
    "owner": "root",
    "size": 35,
    "src": "/home/sangjunyun_1984/.ansible/tmp/ansible-tmp-1584880181.3114061-33928614051676/source",
    "state": "file",
    "uid": 0
}
...
``` 

이 명령을 실행하면 remote host에 "/etc/sudoers.d/example-user" 파일이 생성되고
해당 파일에 "example-user ALL=(ALL) NOPASSWD:ALL" 내용이 추가됩니다.
이 내용은 "example-user"에게 패스워드 없이 sudo를 실행할 수 있는 권한을 부여한다는 의미입니다.

이제 sudo 권한을 확인해보겠습니다. 먼저 "example-user"로 다시 로그인합니다.
```
default-user@ansible-host:~$ su - example-user 
Password: 
```

그리고 "root" 계정으로 로그인하는 sudo 명령을 실행해보겠습니다.
패스워드 없이 root 계정으로 로그인된 것을 확인할 수 있습니다.
```
example-user@ansible-host:~$ sudo su - root

root@ansible-host:~$
```

## 맺음말

지금까지 Ansible 명령을 사용하여 계정 생성 작업을 자동화하는 방법을 알아봤습니다.

Ansible 명령으로 간단한 작업들을 수행하는 것은 편리합니다.
하지만 여러 가지 작업들을 수행해야하는 경우 Ansible 명령만으로 부족한 부분이 있습니다.

이런 경우에 Ansible Playbook을 활용할 수 있습니다.
이후에 Ansible Playbook이 무엇이고 어떻게 사용하는지에 대해서도 한번 살펴 보겠습니다.




