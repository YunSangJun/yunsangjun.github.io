---
layout: post
title:  "Spinnaker 설정 #2 사용자 인증/인가"
author: sj
date: 2018-08-16
categories: spinnaker
tags:
- spinnaker
- kubernetes
- cicd
- devops
---

이 페이지는 Spinnaker에 사용자 인증/인가 기능을 설정하는 방법에 대해 설명합니다.

자세한 내용은 Spinnaker 공식 가이드 문서의 [Security](https://www.spinnaker.io/setup/security/)을 참고하세요.

## 인증(Authentication)

### OAuth

OAuth는 인터넷 사용자들이 비밀번호를 제공하지 않고 웹사이트 상의 자신들의 정보에 대해 접근 권한을 부여할 수 있는 수단으로서 사용되는 표준입니다.

### OAuth 제공자

편의을 위해 아래와 같은 OAuth 제공자들을 built-in으로 제공합니다. Cliet ID와 Secret 설정을 통해 간편하게 구성할 수 있습니다.

Provider | Halyard value | Provider-Specific Docs
Google Apps for Work / G Suite |	google | [Google Apps for Work / G Suite](https://www.spinnaker.io/setup/security/authentication/oauth/providers/google/)
GitHub | github	| [GitHub Teams](https://help.github.com/articles/authorizing-oauth-apps/)
Azure	| azure	| [Azure](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-protocols-oauth-code)

이 페이지에서는 GitHub를 기준으로 설명하겠습니다.

### 사전 준비

GitHub에서 OAuth App을 생성합니다.

Github > Settings > Developer settings > OAuth Apps > New OAuth App 선택

아래와 같이 정보를 입력합니다.

![](/blog/assets/images/spinnaker/spinnaker-create-oauthapp.png)

* Homepage 및 Callback URL은 [도메인 주소를 통한 대시보드 접속](/blog/spinnaker/2018/08/15/configuration-spinnaker-1.html) 가이드의 Spinnaker ui, api 설정
정보입니다.

### OAuth Provider 및 Client 정보 설정

아래와 같이 Client ID, Secret과 OAuth Provider를 설정합니다.

CLIENT_ID, CLIENT_SECRET은 위에서 생성한 OAuth App 정보에서 얻을 수 있습니다.

```
CLIENT_ID=myClientId
CLIENT_SECRET=myClientSecret
PROVIDER=github

hal config security authn oauth2 edit \
  --client-id $CLIENT_ID \
  --client-secret $CLIENT_SECRET \
  --provider $PROVIDER

hal config security authn oauth2 enable
```

### 적용

Spinnaker를 다시 배포합니다.

```
$ hal deploy apply
```

### 접속

웹 브라우저에서 Spinnaker ui 주소로 접속합니다.

예) http://spinnaker.example.com

아래와 같이 Github 로그인 페이지로 redirect 됩니다.

![](/blog/assets/images/spinnaker/spinnaker-oauth-login.png)

이제 GitHub 계정으로 로그인하면 Spinnaker ui 주소록 접속되는 것을 확인 할 수 있습니다.

![](/blog/assets/images/spinnaker/spinnaker-dashboard.png)
