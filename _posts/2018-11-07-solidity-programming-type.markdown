---
layout: post
title:  "Solidity Programming #4 자료형"
author: sj
date: 2018-11-07
categories: solidity
tags:
- ethereum
- solidity
- smartcontract
---

이 페이지에서는 Solidity의 자료형 대해 살펴보겠습니다.

## 값 타입(Value Types)

### 부울(Boolean)

`Boolean`은 true와 false 두 가지 상수를 사용할 수 있습니다.

#### 연산자

연산자 | 의미
`!`  | not
`&&` | and
`||` | or
`==` | equal
`!=` | not equal

연산자 `||`와 `&&`는 `short-circuiting` 규칙이 적용됩니다.
예를 들어, f(x) || g(y) 연산에서 만약 f(x)가 참이면, g(y)는 실행되지 않습니다.

#### Practice

```
bool a = true;
bool b = false;
```

### 정수(Integers)

`int`는 부호가 있는 정수, `uint`는 부호가 없는 정수에 사용합니다.

uint8 ~ uint256는 부호 없는 8 ~ 256 비트 정수입니다.
uint 및 int는 각각 uint256 및 int256를 뜻합니다.

#### 연산자

연산자 | 의미
`<=, <, ==, !=, >=, >` | 비교 연산
`&, |, ^(xor), ~(not)` | 비트 연산
`+, -, *, /, % , ** (제곱), << (left shift), >> (right shift)` | 산술 연산

0으로 나누거나 나머지를 계산하면 예외가 발생합니다.

#### Practice

```
int a = 1;
int b = 2;
```

### 고정 소수점 수(Fixed Point Numbers)

<p class="warning-title">경고</p>
<p class="warning-content">
Solidity는 고정 소수점 수를 아직 완벽하게 지원하지 않습니다.
고정 소수점 수를 선언할 수는 있지만 할당할 수는 없습니다.
</p>

### 주소(Address)

주소는 20 바이트(이더리움 주소의 크기)를 보유합니다.
주소 유형은 멤버를 가지며 contract의 기반이 됩니다.

<p class="tip-title">참고</p>
<p class="tip-content">
Solidity 버전 0.5.0부터 contract는 주소 유형에서 파생되지 않지만 명시 적으로 주소로 변환 할 수 있습니다.
</p>

#### 연산자

`<=, <, ==, !=, >= and >`

#### 멤버

멤버 | 타입 | 의미
balance (uint256) | Property | Wei 단위의 주소의 잔고
transfer (uint256 amount) | Function | 주소에 정해진 Wei를 보내고, 실패시에 예외를 발생시킴. 2300 개의 가스를 전달하고 조절할 수 없음.
send(uint256 amount) returns (bool) | Function | 주소에 정해진 Wei를 보내고, 실패시에 false를 반환함. 2300 개의 가스를 전달하고 조절할 수 없음.
call(...) returns (bool) | Function | low-level의 CALL을 발생시키고 실패시에 false를 반환함. 가능한 모든 gas를 전달하고 조절 가능함.
callcode(...) returns (bool) | Function | low-level의 CALLCODE를 발생시키고 실패시에 false를 반환함. 가능한 모든 gas를 전달하고 조절 가능함.
delegatecall(...) returns (bool) | Function | low-level의 DELEGATECALL을 발생시키고 실패시에 false를 반환함. 가능한 모든 gas를 전달하고 조절 가능함.

#### Practice

아래와 같이 `balnace` property를 활용하여 특정 주소의 잔고를 조회 할 수 있고,
`transfer` 함수를 사용하여 ether를 특정 주소로 전송할 수 있습니다.

```
address x = 0x123;
address myAddress = this;
if (x.balance < 10 && myAddress.balance >= 10) x.transfer(10);
```

### 고정 크기 배열(Fixed-size byte arrays)

고정 크기 배열에는 `bytes1, bytes2, bytes3, ..., bytes32` 까지 32개의 타입이 있습니다.
`byte`는 `bytes1`과 동일합니다.

#### 연산자

연산자 | 의미
`<=, <, ==, !=, >=, >` | 비교 연산
`&, |, ^(xor), ~(not) << (left shift), >> (right shift)` | 비트 연산
`x[k]` | 인덱스 접근, x가 bytesI 배열인 경우, x[k]로 k번째 바이트에 접근 가능(read-only)

#### 멤버

멤버 | 타입 | 의미
length | Property | 바이트 배열의 고정 길이를 가져옴(read-only)

#### Practice

```
bytes1 b1 = "a";
bytes4 b4 = "abcd";
```

### 이넘 타입(Enums)

0부터 6까지의 숫자를 입력받아 각각 월요일 부터 일요일까지의 요일을 반환하는 함수가 있다고 가정해보겠습니다.

아래와 같이 입력받은 값에 따라 요일을 반환하는 방식입니다.
이 때 입력받는 값을 비교할 때 숫자 0~6이 아니라 의미있는 값으로 변환하고 싶다면 이넘 타입을 사용할 수 있습니다.

```
pragma solidity ^0.x.x;

contract EnumTest {

    function returnDayInKorean(uint day) public view returns (string) {

        if(day == 0) {
            return "월요일";

        } else if(day == 1) {
            return "화요일";

        } else if(day == 2) {
            return "수요일";

        } else if(day == 3) {
            return "목요일";

        } else if(day == 4) {
            return "금요일";

        } else if(day == 5) {
            return "토요일";

        } else if(day == 6) {
            return "일요일";

        }
    }

}
```

이넘 타입은 `enum` 키워드를 사용하고 값은 일반적으로 대문자로 시작하도록 입력합니다.
이넘 타입에 입력한 `Monday부터 Sunday`값에는 순서대로 `0부터 6`까지의 값이 할당됩니다.
이넘 타입과 정수 타입간에는 서로 변환이 가능한데 변환 시에는 명시적으로 선언해줘야합니다.

`이넘 타입 -> 정수 타입`으로 변환은 `uint(Day.Monday)` 방식으로 선언합니다.

`정수 타입 -> 이넘 타입`으로 변환은 `Day(0)` 방식으로 선언합니다.

```
pragma solidity ^0.x.x;

contract EnumTest {

    enum Day {
        Monday, Tuesday, Wednesday, Thursday, Friday, Saturday, Sunday
    }

    function returnDayInKorean(uint day) public view returns (string) {

        if(day == uint(Day.Monday)) {
            return "월요일";

        } else if(Day(day) == Day.Tuesday) {
            return "화요일";

        } else if(day == uint(Day.Wednesday)) {
            return "수요일";

        } else if(day == uint(Day.Thursday)) {
            return "목요일";

        } else if(day == uint(Day.Friday)) {
            return "금요일";

        } else if(day == uint(Day.Saturday)) {
            return "토요일";

        } else if(day == uint(Day.Sunday)) {
            return "일요일";

        }
    }

}
```

## 참조 타입(Reference Types)

### 배열(Arrays)

배열에는 컴파일 시 크기가 결정되는 `정적 배열`과 프로그램 실행 시 동적으로 크기가 결정되는 `동적 배열`이 있습니다.

#### 정적 배열(Static Array)

```
//선언
uint8[5] odd = [1, 3, 5, 7, 9];

//인덱싱. odd 배열의 두 번째 element에 접근
odd[1];

//element에 값 할당. odd 배열의 두 번째 element 값 변경
odd[1] = 2;

//배열의 크기 조회. odd 배열의 크기 조회
odd.length;
```

#### 동적 배열(Dynamic Array)

동적 배열에는 스토리지 배열(Storage Array)과 메모리 배열(Memory Array)이 있습니다.

- 동적 스토리지 배열(Dynamic Storage Array)

    배열을 선언할 때 크기를 지정하지 않으면 동적 스토리지 배열로 생성됩니다. 동적 스토리지는 배열의 멤버인
    `length`와 `push`를 사용해서 크기를 조절할 수 있습니다.

    ```
    //선언
    uint[] odd;

    //element에 값 할당. odd 배열의 첫 번째 element에 숫자 1 할당
    odd[0] = 1;
    odd.push(3);

    //인덱싱. odd 배열의 첫 번째 element에 접근
    odd[0];

    //배열의 크기 조회. odd 배열의 크기 조회
    odd.length;

    //배열의 크기 조절. odd 배열의 크기 조절
    odd.length = 5;
    ```

- 동적 메모리 배열(Dynamic Memory Array)

    동적 메모리 배열은 키워드 `new`를 사용하여 생성합니다. 동적 스토리지 배열과 다르게 배열의 크기가
    한번 결정되면 변경할 수 없고, 멤버인 `length`와 `push`를 사용할 수 없습니다.

    ```
    //선언
    uint[] odd = new uint[](5);

    //element에 값 할당. odd 배열의 첫 번째 element에 숫자 1 할당
    odd[0] = 1;

    //인덱싱. odd 배열의 첫 번째 element에 접근
    odd[0];

    //배열의 크기 조회. odd 배열의 크기 조회
    odd.length;
    ```

동적 스토리지 배열(Storage Array)과 메모리 배열(Memory Array) 비교

항목 | 동적 스토리지 배열 | 동적 메모리 배열
push 사용 | O | X
배열 크기 조회 | O | O
배열 크기 조절 | O | X

### 바이트 배열(bytes)

데이터의 크기를 미리 알 수 없는 경우 바이트 배열 `bytes`을 사용하고 크기를 알고 있는 경우
고정 크기 배열(Fixed-size byte arrays)인 `bytes1 ~ bytes32` 중 하나를 사용하는 것이 좋습니다
`bytes1 ~ bytes32`가 `bytes` 보다 자원을 덜 사용하기 때문입니다.

```
//선언
bytes memory b = new bytes(10);

//element에 값 할당. b 배열의 첫 번째 element에 바이트 할당
b[0] = byte(65);

//인덱싱. b 배열의 첫 번째 element에 접근
b[0];

//배열의 크기 조회. b 배열의 크기 조회
b.length;
```

### 문자열(String)

문자열은 `UTF-8`로 인코딩된 문자열을 저장하기 위한 동적 크기 배열입니다.

문자열은 바이트 배열(bytes)과 동일하지만 길이나 인덱스 접근을 허용하지 않습니다.

```
string memory str = "foo"
string memory str = 'bar'
```

## 매핑(Mappings)

매핑은 다른 프로그래밍 언어에서의 map 또는 dictionary와 같은 해시테이블(hash tables) 형태의 자료 구조입니다.

`mapping(KeyType => ValueType)`과 같은 형태로 선언하며 키 타입에는(KeyType)에는 매핑, 동적 크기 배열,
컨트랙트, 스트럭트, 이넘을 사용할 수 없습니다. 값 타입(ValueType)에는 모든 타입을 사용할 수 있습니다.

```
pragma solidity ^0.x.x;

contract MappingExample {
    mapping(address => uint) public balances;

    function update(uint newBalance) public {
        balances[msg.sender] = newBalance;
    }
}

contract MappingUser {
    function f() public returns (uint) {
        MappingExample m = new MappingExample();
        m.update(100);
        return m.balances(this);
    }
}
```

위 예제에서는 `balances` 상태 변수를 매핑 타입으로 선언했습니다. 키 타입은 주소이고 값은 정수입니다.
