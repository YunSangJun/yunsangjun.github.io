---
layout: post
title:  "Spinnaker 설정 #3 CI 시스템 연계"
author: 윤상준
date: 2018-08-18
categories: spinnaker
tags:
- spinnaker
- kubernetes
- cicd
- devops
---

이 페이지는 Spinnaker와 CI 시스템을 연계하는 방법에 대해 설명합니다.

자세한 내용은 Spinnaker 공식 가이드 문서의 [Add Your CI system](https://www.spinnaker.io/setup/ci/jenkins/)을 참고하세요.

## Jenkins

### Jenkins master 추가

1. Jenkins 활성화

    ```
    hal config ci jenkins enable
    ```

2. Jenkins master 추가

    ```
    export BASEURL=http://jenkins.example.com
    export USERNAME=example
    export USERNAME=example

    echo $PASSWORD | hal config ci jenkins master add my-jenkins-master \
        --address $BASEURL \
        --username $USERNAME \
        --password
    ```

3. 적용

    ```
    hal deploy apply
    ```

### CSRF protection 설정

<p class="tip-title">참고</p>
<p class="tip-content">
Jenkins CSRF protection in Igor is only supported for Jenkins 2.x.
</p>

1. csrf 활성화

    ```
    hal config ci jenkins master edit MASTER --csrf true
    ```

2. 적용

    ```
    hal deploy apply
    ```

3. Jenkins에서 CSRF protection 활성화

    - Manage Jenkins > Configure Global Security, `Prevent Cross Site Request Forgery exploits` 선택

    - Crumb Algorithm, `Default Crumb Issuer` 선택
