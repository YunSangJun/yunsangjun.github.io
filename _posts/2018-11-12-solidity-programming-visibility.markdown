---
layout: post
title:  "Solidity Programming #6 가시성(Visibility)"
author: 윤상준
date: 2018-11-12
categories: solidity
tags:
- ethereum
- solidity
- smartcontract
---

이 페이지에서는 Solidity의 가시성(Visibility)에 대해 살펴보겠습니다.

## 가시성(Visibility)

Solidity에는 두 가지의 함수 호출(function call)이 있습니다.
실제 EVM 호출을 만들지 않는 내부(internal) 함수 호출과 EVM 함수 호출을 만드는 외부(external) 함수 호출입니다.

### 가시성의 종류

함수와 상태 변수(state variables)는 아래와 같이 `external`, `public`, `internal`, `private` 4가지로 선언할 수 있습니다.

함수(function)의 기본 값은 `public`입니다.

상태 변수(state variables)는 `external` 사용이 불가능하고 기본값은 `internal`입니다.

- external: contract 외부에서만 호출 가능합니다.
- public: contract 내부 및 외부애서 호출 가능합니다. 상태 변수를 public으로 선언하면 자동으로 getter 함수가 생성됩니다.
- internal: contract 내부 및 상속받은 contract에서 호출 가능합니다.
- private: contract 내부에서만 호출 가능합니다.

<p class="tip-title">참고</p>
<p class="tip-content">
contract 내부의 모든 것은 외부에 노출됩니다. private으로 선언하여 다른 contract가 정보를 접근하고 수정하는 것을 막을 수는 있지만
여전히 전 세계의 블록체인(Block Chain)에서 볼 수 있습니다. Ethereum 환경에서 smart contract 개발은 실제 현금과
교환할 수 있는 암호화폐를 다루기 때문에 일반 애플리케이션을 개발할 때보다 더 주의해야한다는 것을 명심해야합니다.
</p>

### Practice

가시성은 상태 변수의 자료형 뒤에 선언하고 함수의 매개 변수 및 반환 값 사이에 선언합니다.

```
pragma solidity ^0.x.x;

contract C {
    function f(uint a) private pure returns (uint b) { return a + 1; }
    function setData(uint a) internal { data = a; }
    uint public data;
}
```

아래 예제에서 contract `D`는 `public`으로 선언된 `c.getData()` 함수를 호출해 `data` 상태 변수의 값을 가져올 수 있습니다.
하지만 `private`으로 선언된 `f` 함수는 호출할 수 없습니다.

Contract `C`를 상속받은 `E`는 `internal`로 선언된 `compute` 함수를 호출할 수 있습니다.

```
pragma solidity ^0.x.x;

contract C {
    uint private data;

    function f(uint a) private returns(uint b) { return a + 1; }
    function setData(uint a) public { data = a; }
    function getData() public returns(uint) { return data; }
    function compute(uint a, uint b) internal returns (uint) { return a+b; }
}

contract D {
    function readData() public {
        C c = new C();
        uint local = c.f(7); // error: member `f` is not visible
        c.setData(3);
        local = c.getData();
        local = c.compute(3, 5); // error: member `compute` is not visible
    }
}

contract E is C {
    function g() public {
        C c = new C();
        uint val = compute(3, 5); // access to internal member (from derived to parent contract)
    }
}
```

## Getter 함수

상태 변수를 `public`으로 선언하면 컴파일러가 자동으로 getter 함수를 생성합니다.

아래 예제에서 컴파일러는 `public`으로 선언된 상태 변수 `data`에 대한 getter 함수를 생성합니다.
getter 함수는 입력받는 매개변수가 없으며 반환 값으로 `uint` 형의 상태 변수 `data`의 값을 반환합니다.

```
pragma solidity ^0.4.0;

contract C {
    uint public data = 42;

    //컴파일 시 내부적으로 생성되는 getter 함수.(실제 코드 상에는 존재하지 않음)
    function data() public returns(uint) {
      return data;
    }
}

contract Caller {
    C c = new C();

    function f() public {
        uint local = c.data();
    }
}
```

상태 변수 `data`는 `public`으로 선언했으므로 외부에서 접근할 수 있습니다.
만약 내부적으로(접근자 `this` 사용 안함) 접근하면 상태 변수로 간주되고, 외부적으로(접근자 `this` 사용) 접근하면 getter 함수로 간주됩니다.

```
pragma solidity ^0.4.0;

contract C {
    uint public data;

    function x() public {
        data = 3; // 내부 접근
        uint val = this.data(); // 외부 접근
    }
}
```
