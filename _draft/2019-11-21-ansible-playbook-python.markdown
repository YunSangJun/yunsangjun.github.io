---
layout: post
title:  "Ansible Playbook - Python 연동"
author: sj
date: 2019-11-21
categories: automation
tags:
- ansible
- ansibleplaybook
- automation
- devops
---

## Overview

자동화 코드를 작성하다보면 문자열을 다루어야 할 경우가 많습니다.
(예를 들어 버전 정보를 취합하는 경우 특정 기호를 기준으로 문자열을 분리해야 한다던지..)

이런 경우 Playbook 문법에서도 어느 정도 지원을 하지만 익숙하지도 않고
기본으로 지원하는 문자열 함수가 많지 않아 코드 작성에 어려움을 겪을 수 있습니다.

이런 경우 Playbook 코드에서 python을 연동해 문자열을 좀 더 쉽게 처리할 수 있습니다.
Python은 러닝커브가 상대적으로 낮고 풍부한 문자열 처리 함수를 기본적으로 제공합니다.

만약 Ansible Playbook을 처음 접하는 분이라면 
[Ansible Playbook 시작하기](/automation/2019/11/07/ansible-playbook-start.html)에서 
기본적인 개념과 사용 방법에 대해 살펴보는것이 좋습니다.

