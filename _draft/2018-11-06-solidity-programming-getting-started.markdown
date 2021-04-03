---
layout: post
title:  "Solidity Programming #1 Getting started"
author: sj
date: 2018-11-06
categories: solidity
tags:
- ethereum
- geth
- smartcontract
- dapp
- mining
---

Solidity는 Ethereum 환경에서 Smart Contract를 프로그래밍하는데 사용하는 언어 중 하나입니다.

이 페이지에서는 Solidity 언어로 Hello world를 출력하는 Smart Contract를 작성해보겠습니다.

## Solidity IDE 실행

Ethereum에서는 Solidity 프로그래밍을 하기 위한 개발환경으로 Web IDE를 제공합니다.

웹 브라우저에서 [Solidity IDE](http://remix.ethereum.org)를 접속합니다.

## 컴파일러 버전 선택

IDE 오른쪽 > Compile 탭에서 컴파일러 버전을 선택합니다.

이 페이지에서는 `0.4.24` 버전을 선택하겠습니다.

![](/assets/images/solidity/select-compiler-version.png)

## 실행 환경 선택

IDE 오른쪽 > Run 탭에서 실행 환경을 선택합니다.

일반적으로 `Javascript VM`을 선택합니다.

![](/assets/images/solidity/select-running-environment.png)

## 파일 생성

화면 상단의 `+` 버튼을 클릭합니다.

![](/assets/images/solidity/create-file1.png)

파일명을 입력하고 OK 버튼을 선택해 파일을 생성합니다.

![](/assets/images/solidity/create-file2.png)

## Smart Contract 작성

아래와 같이 Hello world를 출력하는 Smart Contract를 작성합니다.

컴파일러 버전은 위에서 선택한 `0.4.24` 버전을 선언합니다.

```
pragma solidity ^0.4.24;

contract HelloWorld {

    function printHello() returns (string) {

        return "Hello world";

    }

}
```

문법을 자세히 보면 Javascript와 유사한 것을 알 수 있습니다.

문법에 대한 상세한 내용은 추후 자세히 다루겠습니다.

## Smart Contract 배포 및 실행

`ctrl + s`로 파일을 저장한 후 오른쪽 Run 탭을 확인해보면 `Hello World` Smart Contract가 생성
된 것을 확인할 수 있습니다.

![](/assets/images/solidity/deploy-smart-contract.png)

여기서 `Deploy`를 선택하면 Smart Contract가 배포됩니다.

하단의 콘솔 로그를 확인하면 `Hello world` 문자열이 출력된 것을 확인할 수 있습니다.

![](/assets/images/solidity/console-log.png)
