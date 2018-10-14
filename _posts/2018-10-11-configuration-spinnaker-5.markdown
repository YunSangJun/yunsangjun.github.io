---
layout: post
title:  "Spinnaker 설정 #5 Artifact 설정"
author: 윤상준
date: 2018-10-11
categories: spinnaker
---

이 페이지는 Spinnaker에 Artifact account를 설정하는 방법에 대해 설명합니다.

자세한 내용은 Spinnaker 공식 가이드 문서의 [Configure Artifacts](https://www.spinnaker.io/setup/artifacts/)을 참고하세요.

## GitHub artifact account 설정

Spinnaker에서는 GitHub 저장소에서 배포할 리소스를 가져올 수 있습니다.
이를 Artifact라고 부릅니다.

아래 가이드를 따라서 Spinnaker에 GitHub artifact account를 설정할 수 있습니다.

### 사전 준비

#### credentials 다운로드

[Personal access tokens](https://github.com/settings/tokens) 페이지에서 토크을 생성합니다.

이때 필요한 scope는 `repo` scope 입니다.

Halyard에서 토큰을 읽을 수 있도록 아래와 같이 설정합니다.

```
export TOKEN=EXAMPLE_TOKEN
export TOKEN_FILE=EXAMPLE_TOKEN_FILE
echo $TOKEN > $TOKEN_FILE
```

### Artifact 설정 편집

`ARTIFACT_ACCOUNT_NAME`에 GitHub artifact 계정을 설정합니다.

```
export ARTIFACT_ACCOUNT_NAME=my-github-artifact-account
```

artifact 설정을 활성화합니다.

```
hal config features edit --artifacts true
hal config artifact github enable
```
artifact 계정을 설정합니다.

```
hal config artifact github account add $ARTIFACT_ACCOUNT_NAME \
    --token-file $TOKEN_FILE
```

변경 사항을 반영합니다.
```
hal deploy apply
```
