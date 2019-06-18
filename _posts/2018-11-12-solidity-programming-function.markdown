---
layout: post
title:  "Solidity Programming #5 함수 호출"
author: 윤상준
date: 2018-11-12
categories: solidity
tags:
- ethereum
- solidity
- smartcontract
---

이 페이지에서는 Solidity의 함수 호출(function call)에 대해 살펴보겠습니다.

## 내부 함수 호출(internal function call)

Contract 내부에 있는 함수를 그대로 호출하는 경우를 내부 함수 호출이라하고 합니다.
내부 함수 호출은 EVM 호출을 만들지 않고 단순 점프로 변환됩니다. 이 때 함수의 매개변수는 EVM의 메모리에 저장됩니다.

```
pragma solidity ^0.x.x;

contract C {
    function sum(uint x, uint y) public returns(uint) {
        return x + y;
    }

    function internalFunctionCall(uint x, uint y, uint z) public returns(uint) {
        //내부 함수 호출
        return sum(x, y) + z;
    }
}
```

## 외부 함수 호출(external function call)

다른 contract의 함수를 호출하는 것을 외부 함수 호출이라하고 합니다.
또 같은 contract 내부의 함수라도 접근자 `this`를 붙이면 외부 함수 호출로 처리됩니다.
외부 함수 호출은 EVM 함수 호출을 생성합니다. 이 때 함수의 매개변수는 EVM의 메모리가 아니라 콜 데이터 영역에 저장됩니다.

```
pragma solidity ^0.x.x;

contract C {
    function sum(uint x, uint y) public returns(uint) {
        return x + y;
    }

    function externalFunctionCall(uint x, uint y, uint z) public returns(uint) {
        //외부 함수 호출
        return this.sum(x, y) + z;
    }
}

contract D {
    function externalFunctionCall(uint x, uint y, uint z) public returns(uint) {
        //외부 함수 호출
        C c = new C();
        return c.sum(x, y) + z;
    }
}
```

<p class="tip-title">참고</p>
<p class="tip-content">
가시성이 public인 함수는 외부 및 내부 호출이 둘 다 가능합니다. 따라서 함수를 pulbic으로 선언하면
매개변수를 메모리로 복사하게 됩니다. 만약 매개변수의 데이터 크기가 크다면 메모리를 많이 사용하므로
이에 대한 비용 또한 많이 지불해야합니다. 따라서 외부에서만 호출하고 내부에서 호출하지 않는 함수라면
external로 선언하는 것이 좋습니다. 함수를 external로 선언하면 내부에서 호출이 불가능하므로 매개변수를
메모리에 복사하지 않습니다.
</p>

외부 함수 호출 시 `value`를 사용해 전송하는 이더 값을 설정할 수 있고 `gas`를 사용해 사용할 가스를 설정할 수 있습니다.

```
pragma solidity ^0.x.x;

contract C {
    function sum(uint x, uint y) public payable returns(uint) {
        return x + y;
    }
}

contract D {
    function externalFunctionCall(uint x, uint y, uint z) public returns(uint) {
        //외부 함수 호출
        C c = new C();
        return c.sum.value(10).gas(100)(x, y) + z;
    }
}
```
