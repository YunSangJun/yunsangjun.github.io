---
layout: post
title:  "Solidity Programming #7 가변성(Mutability)"
author: sj
date: 2018-11-13
categories: solidity
tags:
- ethereum
- solidity
- smartcontract
---

이 페이지에서는 Solidity의 함수 상태 가변성(Mutability)에 대해 살펴보겠습니다.

## 함수의 상태 가변성

함수의 상태 가변성은 contract에 선언된 상태 변수를 함수에서 읽거나 쓸 수 있는지를 제어하기 위한 용도로 사용됩니다.

먄약 함수의 상태 가변성을 위반한 경우가 있는 경우 컴파일 오류가 발생합니다.

함수의 상태 가변성은 키워드는 아래와 같이 `pure`, `view`, `payable` 세 가지가 있습니다.
각 키워드별로 읽기, 쓰기와 송금에 대한 권한이 다르게 설정됩니다.
키워드를 설정하지 않으면 기본 값이 설정됩니다.

| 상태 가변성 | 읽기 | 쓰기 | 송금 |
| pure    | X | X | X |
| view    | O | X | X |
| payable | O | O | O |
| 기본     | O | O | X |

실제 예제를 통해서 어떤 방식으로 동작하는지 살펴보겠습니다.

Contract `C`의 `getData` 함수에서는 `data` 상태 변수의 값을 읽어와 결과 값으로 반환하는 기능을
수행합니다.

```
pragma solidity ^0.4.24;

contract C {
    uint public data = 1;

    function getData() public pure returns(uint){
        return data;
    }
}
```

함수 내에서 상태 변수에 접근해 읽기 동작을 수행해야 하는데 읽기 동작 수행이 금지된 가변성 키워드 `pure`를 사용하므로
아래 그림과 같은 컴파일 에러가 발생합니다.

![](/blog/assets/images/solidity/mutability1.png)

그렇다면 키워드 `pure`를 삭제하면 어떻게 될까요?

```
pragma solidity ^0.4.24;

contract C {
    uint public data = 1;

    function getData() public returns(uint){
        return data;
    }
}
```

키워드를 설정하지 않았기 때문에 실제 함수 상태 가변성은 기본 값이 설정됩니다. 기본 값에서는 함수에서 상태 변수에
접근해 읽기 동작을 수행할 수 있으므로 컴파일 에러는 발생하지 않지만 경고 메세지가 출력되는 것을 볼 수 있습니다.
경고 메세지의 뜻은 함수의 상태 가변성을 `view`로 제한할 수 있다는 뜻입니다.

![](/blog/assets/images/solidity/mutability2.png)

경고 메시지의 가이드에 따라 함수 상태 가변성 키워드를 `view`로 변경해주면 컴파일이나 경고 메세지가 출력되지 않는
것을 확인할 수 있습니다.

```
pragma solidity ^0.4.24;

contract C {
    uint public data = 1;

    function getData() public view returns(uint){
        return data;
    }
}
```
