---
layout: post
title:  "Spinnaker 추가 설정 #2 사용자 인증"
author: 윤상준
date: 2018-08-16
categories: spinnaker
---

이 페이지는 Spinnaker에 사용자 인증 기능을 추가하는 방법에 대해 설명합니다.

자세한 내용은 Spinnaker 공식 가이드 문서의 [OAuth 2.0](https://www.spinnaker.io/setup/security/authentication/oauth/)을 참고하세요.

## OAuth 2.0

OAuth는 인터넷 사용자들이 비밀번호를 제공하지 않고 웹사이트 상의 자신들의 정보에 대해 접근 권한을 부여할 수 있는 수단으로서 사용되는 표준입니다.

### OAuth 제공자

#### Build-in

편의을 위해 아래와 같은 OAuth 제공자들을 built-in으로 제공합니다. Cliet ID와 Secret 설정을 통해 간편하게 구성할 수 있습니다.

Provider | Halyard value | Provider-Specific Docs
Google Apps for Work / G Suite |	google | [Google Apps for Work / G Suite](https://www.spinnaker.io/setup/security/authentication/oauth/providers/google/)
GitHub | github	| [GitHub Teams](https://help.github.com/articles/authorizing-oauth-apps/)
Azure	| azure	| [Azure](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-protocols-oauth-code)

아래와 같이 Client ID, Secret과 OAuth Provider를 설정합니다.

```
CLIENT_ID=myClientId
CLIENT_SECRET=myClientSecret
PROVIDER=google|github|azure

hal config security authn oauth2 edit \
  --client-id $CLIENT_ID \
  --client-secret $CLIENT_SECRET \
  --provider $PROVIDER

hal config security authn oauth2 enable
```

Spinnaker를 다시 배포합니다.

```
$ hal deploy apply
```
