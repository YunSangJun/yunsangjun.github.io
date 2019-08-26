---
layout: post
title:  "Solidity Programming #3 Contract의 구조"
author: sj
date: 2018-11-07
categories: solidity
tags:
- ethereum
- solidity
- smartcontract
---

이 페이지에서는 Solidity의 Contract의 구조에 대해 살펴보겠습니다.

Solidity에서 contract는 객체 지향(object-oriented) 언어의 클래스(Class)와 유사합니다.
각각의 contract는 상태 변수(state variables), 함수(function), 이벤트(events), 구조체(structure)를 포함할 수 있습니다.
또한 contract는 다른 contract를 상속받을 수 있습니다.

## 상태 변수(state variables)

상태 변수는 contract의 저장소(Storage)에 영구적으로 저장되는 값입니다.

```
pragma solidity ^0.x.x;

contract SimpleStorage {
    uint storedData; // 상태 변수
    // ...
}
```

상태 변수의 유형에 대한 내용은 [자료형](/blog/solidity/2018/11/07/solidity-programming-type.html)을 참고하세요.

상태 변수의 [가시성](/blog/solidity/2018/11/12/solidity-programming-visibility.html)은 상태 변수에 대한 접근을 제어하는 용도로 사용합니다.

## 함수(function)

함수는 contract 내에서 실행 가능한 코드 단위입니다.

```
pragma solidity ^0.x.x;

contract SimpleAuction {
    function bid() public payable { // 함수
        // ...
    }
}
```

Solidity에서 [함수 호출](/blog/solidity/2018/11/12/solidity-programming-function.html) 방식은 내부 호출(internal function call)과 외부 호출(external function call)이 있습니다.

함수의 [가시성](/blog/solidity/2018/11/12/solidity-programming-visibility.html)은 함수에 대한 접근을 제어하는 용도로 사용합니다.

함수의 [가변성](/blog/solidity/2018/11/13/solidity-programming-mutability.html)은 함수에서 contract에 정의된 상태 변수에 접근하는 것을 제어하는 용도로 사용합니다.
