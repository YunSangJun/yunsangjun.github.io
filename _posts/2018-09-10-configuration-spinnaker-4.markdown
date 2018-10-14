---
layout: post
title:  "Spinnaker 설정 #4 GitHub Webhooks 설정"
author: 윤상준
date: 2018-09-10
categories: spinnaker
---

이 페이지는 Spinnaker에 GitHub Webhook을 설정하는 방법에 대해 설명합니다.

자세한 내용은 Spinnaker 공식 가이드 문서의 [Configuring GitHub Webhooks](https://www.spinnaker.io/setup/triggers/github/)을 참고하세요.

## GitHub Webhook 설정

Spinnaker는 GitHub 저장소의 변경사항을 감지할 수 있습니다.

이를 위해서는 Webhook 설정이 필요합니다.

### 사전 준비

아래와 같이 `services.gate.baseUrl`을 확인합니다.

```
$ cat ~/.hal/default/staging/spinnaker.yml
services:
...
gate:
  ...
  baseUrl: http://spinnaker-api.example.com
```

### Webhook 설정

Repository > Settings > Webhooks > Add Webhook 선택

아래와 같이 정보를 입력한 후 `Add webhook` 버튼을 선택합니다.

* Payload URL은 위에서 확인한 `services.gate.baseUrl` + `/webhooks/git/github` 입니다.

![](/blog/assets/images/spinnaker/spinnaker-webhook.png)

이제 GitHub 저장소에 push event가 발생하면 Spinnaker에서 이벤트를 수신하게됩니다.
