---
layout: post
title:  "Ansible Playbook 응용"
author: sj
date: 2019-11-18
categories: automation
tags:
- ansible
- ansibleplaybook
- automation
- devops
---

## Overview

이 문서에서는 예제를 통해 Ansible Playbook의 아래와 같은 기능에 대해 알아보겠습니다.

- 변수(register)
- 메세지 출력(debug.msg)
- 조건문(when)
- 파일 생성하기(file)
- 파일에 결과 append하기(lineinfile)
- 한번만 실행하기(run_once)
- localhost에서 실행하기(delegate_to)

만약 Ansible Playbook을 처음 접하는 분이라면 
[Ansible Playbook 시작하기](/automation/2019/11/07/ansible-playbook-start.html)에서 
기본적인 개념과 사용 방법에 대해 살펴보는것이 좋습니다.

## 변수(register) 저장 및 출력

Playbook에서 task를 수행 시 어떤 결과를 저장하고 다음 단계에서 이 값을 사용하고 싶은 경우가 있습니다.
이때 "register" 모듈을 사용해 변수에 저장할 수 있습니다.

간단한 예제를 통해 자세히 알아보겠습니다.

"example.yaml" 이라는 파일을 생성하고 아래와 같이 작성해보겠습니다.

- "Check OS version" task 추가
  - shell: shell 모듈 선언 및 OS version을 읽어오는 스크립트 작성
  - register: register 모듈 선언 및 결과 값을 저장할 변수명 입력 

```yaml
---
{% raw %}
- name: Example
  hosts: "{{ HOST_NAME }}"
  become: true
  tasks:
  - name: Check OS version
    shell: cat /etc/*rele* |grep VERSION=
    register: result
{% endraw %}
```

변수에 저장을 했으니 다음으로 잘 저장되었는지 출력을 해보겠습니다.
"example.yaml" 파일에 이어서 작성하겠습니다.

- "Print result variable" task 추가
  - debug: debug 모듈 선언
  - msg: msg 파라미터에 result 변수의 "stdout"을 입력

```yaml
---
{% raw %}
- name: Example
  hosts: "{{ HOST_NAME }}"
  become: true
  tasks:
  - name: Check OS version
    shell: cat /etc/*rele* |grep VERSION=
    register: result
  - name: Print result variable
    debug:
      msg:
        - "{{ result.stdout }}"
{% endraw %}
```

이제 작성한 playbook을 실행해 보겠습니다.

- "i" 옵션: Remote host의 정보를 가지고 있는 inventory 파일 경로를 지정
- "--extra-vars" 옵션: Playbook에 전달할 인자 값
  - HOST_NAME: Remote host 이름

```bash
default-user@ansible-host:~$ ansible-playbook -i [path/to/inventory] example.yaml \
--extra-vars "HOST_NAME=[HOST_NAME]"

PLAY [Example] *****************************************************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************************************************
ok: [host02.example.com]
ok: [host03.example.com]
ok: [host01.example.com]

TASK [Check OS version] ******************************************************************************************************************************************
changed: [host02.example.com]
changed: [host03.example.com]
changed: [host01.example.com]

TASK [Print result variable] *************************************************************************************************************************************
ok: [host01.example.com] => {
    "msg": [
        "VERSION=\"16.04.6 LTS (Xenial Xerus)\""
    ]
}
ok: [host02.example.com] => {
    "msg": [
        "VERSION=\"16.04.6 LTS (Xenial Xerus)\""
    ]
}
ok: [host03.example.com] => {
    "msg": [
        "VERSION=\"16.04.6 LTS (Xenial Xerus)\""
    ]
}

PLAY RECAP **************************************************************************************************************************************** 
host01.example.com         : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
host02.example.com         : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
host03.example.com         : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

"Print result variable" task의 결과를 확인해보면 remote host의 OS 버전 정보를 확인할 수 있습니다.
이런 식으로 변수에 결과를 저장하고 다른 task에서 해당 변수의 값을 사용할 수 있습니다.

<p class="tip-title">참고</p>
<p class="tip-content">
Task의 반환 값에는 아래와 같이 실행과 관련된 부가적인 정보들이 있습니다.<br/>
반환 값에서 수행 결과 값만 가져오고 싶은 경우 "stdout" 또는 "stdout_lines"을 사용하면됩니다.<br/>
"stdout_lines"은 결과 값이 여러줄인 경우 사용하기 더 편리합니다.
</p>

```bash
ok: [host01.example.com] => {
    "msg": [
        {
            "changed": true,
            "cmd": "cat /etc/*rele* |grep VERSION=",
            "delta": "0:00:00.009317",
            "end": "2019-11-18 13:03:35.726500",
            "failed": false,
            "rc": 0,
            "start": "2019-11-18 13:03:35.717183",
            "stderr": "",
            "stderr_lines": [],
            "stdout": "VERSION=\"16.04.6 LTS (Xenial Xerus)\"",
            "stdout_lines": [
                "VERSION=\"16.04.6 LTS (Xenial Xerus)\""
            ]
        }
    ]
}
```
반환 값에 대한 자세한 정보는 
[Return Values](https://docs.ansible.com/ansible/latest/reference_appendices/common_return_values.html)
페이지를 참고하세요.

## 조건문(when)

이번에는 조건문에 대해 알아보겠습니다.
Task를 특정 조건일 때만 수행하고 싶은 경우 "when" 모듈을 사용하면됩니다.

간단한 예제를 통해 자세히 알아보겠습니다.
"example.yaml" 파일에 이어서 작성하겠습니다.

기존에 작성한 "Print result variable" task에 "when" 모듈을 추가하겠습니다.
그리고 "result.stdout"에 "18.04"라는 문자열이 있는 경우에만 출력하도록 조건을 추가합니다.

```yaml
---
{% raw %}
- name: Example
  hosts: "{{ HOST_NAME }}"
  become: true
  tasks:
  ...
  - name: Print result variable
    debug:
      msg:
        - "{{ result.stdout }}"
    when: result.stdout.find('18.04') != -1
{% endraw %}
```

이제 다시 한번 playbook을 실행해 보겠습니다.

```bash
default-user@ansible-host:~$ ansible-playbook -i [path/to/inventory] example.yaml \
--extra-vars "HOST_NAME=[HOST_NAME]"

...

TASK [Print result variable] ***************************************************************************************************************************************
skipping: [host01.example.com]
skipping: [host02.example.com]
skipping: [host03.example.com]

...
```

"result.stdout"에는 "18.04"라는 문자열이 없으로 skipping 메세지와 함께 아무것도 출력되지 않습니다.
"16.04"로 조건을 변경하면 다시 정상적으로 출력되는 것을 확인할 수 있습니다.

이렇게 "when" 모듈에 원하는 조건을 넣어주면 해당 조건을 만족할 경우에만 task가 수행되도록 할 수 있습니다.

## 파일 생성하기(file)

이번에는 파일을 생성해보겠습니다.
"example.yaml" 파일에 이어서 작성하겠습니다.

- "Create a file" task 추가
  - file: file 모듈 선언
    - path: 파일을 생성할 경로 입력
    - state: "touch" 옵션 선언. 파일이 존재하지 않을 경우에만 생성
  - delegate_to: "localhost" 옵션 선언. Ansible host에서만 실행됨
  - run_once: "yes"인 경우 1회만 실행됨

특이한 점은 파일 생성 task를 Ansible host에서만 1회만 실행하기 위해 "delegate_to"와 "run_once" 모듈을 사용했습니다.

또한 특정 조건인 경우만 실행되도록 위에서 작성한 when 모듈도 추가했습니다.

```yaml
---
{% raw %}
- name: Example
  hosts: "{{ HOST_NAME }}"
  become: true
  tasks:
  ...
  - name: Print result variable
    debug:
      msg:
        - "{{ result.stdout }}"
    when: result.stdout.find('16.04') != -1
  - name: Create a file
    file:
      path: "/tmp/example.txt"
      state: touch
    delegate_to: localhost
    run_once: yes
    when: result.stdout.find('16.04') != -1
{% endraw %}
```

이제 다시 한번 playbook을 실행해 보겠습니다.

```bash
default-user@ansible-host:~$ ansible-playbook -i [path/to/inventory] example.yaml \
--extra-vars "HOST_NAME=[HOST_NAME]"

...

TASK [Create a file] ***********************************************************************************************************************************************
changed: [ansible-host -> localhost]

...
```

"Create a file" task 결과를 확인해보면 ansible-host에서만 실행된 것을 확인할 수 있습니다.

실제로 파일이 잘 생성되었는지도 확인해보겠습니다.
"example.txt" 파일이 "tmp" 디렉토리에 정상적으로 생성되었습니다.

```bash
default-user@ansible-host:~$ cd /tmp
default-user@ansible-host:~$ ls
example.txt ...
```

## 파일에 결과 추가하기(lineinfile)

이번에는 결과 값을 파일에 추가하는 방법을 알아보겠습니다.
"example.yaml" 파일에 이어서 작성하겠습니다.

- "Append result" task 추가
  - lineinfile: lineinfile 모듈 선언
    - line: 파일에 추가할 값 입력
    - dest: 파일 경로 입력
    - insertafter: "EOF" 옵션 선언. 파일의 마지막에 추가
  - delegate_to: "localhost" 옵션 선언. Ansible host에서만 실행됨

특이한 점은 "run_once" 모듈을 선언하지 않았습니다.
각각의 Remote host에서 결과값을 가져와야하기 때문입니다.

{% raw %}"{{ ansible_hostname }}"{% endraw %}과 같이 선언한 부분은
결과값을 가져온 remote host 이름을 추가하기 위해서입니다.

"result.stdout"을 그대로 출력하지 않고 실제 버전 정보만 가져오기 위해 문자열 편집했습니다.

```yaml
---
{% raw %}
- name: Example
  hosts: "{{ HOST_NAME }}"
  become: true
  tasks:
  ...
  - name: Create a file
    file:
      path: "/tmp/example.txt"
      state: touch
    delegate_to: localhost
    run_once: yes
    when: result.stdout.find('16.04') != -1
  - name: Append result
    lineinfile:
      line: "{{ ansible_hostname }} : {{ (result.stdout | regex_replace('\"')).split('=')[1] }}"
      dest: "/tmp/example.txt"
      insertafter: EOF
    delegate_to: localhost
    when: result.stdout.find('16.04') != -1
{% endraw %}
```

이제 다시 한번 playbook을 실행해 보겠습니다.

```bash
default-user@ansible-host:~$ ansible-playbook -i [path/to/inventory] example.yaml \
--extra-vars "HOST_NAME=[HOST_NAME]"

...

TASK [Append result] ***********************************************************************************************************************************************
changed: [host02.example.com -> localhost]
changed: [host03.example.com -> localhost]
changed: [host01.example.com -> localhost]

...
```

"Append result" task의 결과를 확인해보면 각 remote host로 부터 가져온 결과를 localhost(Ansible host)
의 파일에 추가한것을 확인할 수 있습니다.

실제로 파일에 잘 추가되었는지 확인해보겠습니다.
"example.txt" 파일에 각 remote host의 결과가 정상적으로 추가되었습니다.

```bash
default-user@ansible-host:~$ cat /tmp/example.txt 
host01.example.com : 16.04.6 LTS (Xenial Xerus)
host02.example.com : 16.04.6 LTS (Xenial Xerus)
host03.example.com : 16.04.6 LTS (Xenial Xerus)
```

## 맺음말

지금까지 Ansible Playbook의 추가적인 기능에 대해 알아봤습니다.
 
앞으로 자동화 코드를 작성하다보면 문자열을 다루어야 할 경우가 많습니다.

간단한 문자열 처리는 Playbook 내에서 처리할 수 있지만
문법이 익숙하지 않고 지원하는 문자열 함수가 많지 않아 코드 작성에 어려움을 겪을 수 있습니다.

이런 경우 Playbook 코드에서 python을 연동해 문자열을 좀 더 쉽게 처리할 수 있습니다.
Python은 러닝커브가 상대적으로 낮고 풍부한 문자열 처리 함수를 기본적으로 제공합니다.

다음에는 Ansible Playbook과 Python을 연동하는 방법에 대해 알아보겠습니다.
