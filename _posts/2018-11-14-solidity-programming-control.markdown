---
layout: post
title:  "Solidity Programming #8 제어문"
author: 윤상준
date: 2018-11-14
categories: solidity
tags:
- ethereum
- solidity
- smartcontract
---

이 페이지에서는 Solidity의 제어문에 대해 살펴보겠습니다.

Javascript의 `switch` 및 `goto`를 제외한 대부분의 제어문을 Solidity에서 사용할 수 있습니다.
if, else, while, do, for, break, continue, return 등의 제어문을 지원합니다.

## 조건문(if)

```
int a = 3;

if(a > 1) {
  return "a is greater than 1";
} else{
  return "a is less than 1";
}
```

* 주의 : bool 아닌 유형에 대해 유형 변환이 없기 때문에 if(1) {...}이 유효하지 않습니다.

## 반복문(while)

```
int i = 0;
int sum = 0;

while(i<10){
    sum = sum + i;
    i++;
}
```

## 반복문(for)

```
int num = 5;

for(int i=0; i<10; i++){
    if(num % 2 == 0){
        return "Even";
    } else{
        return "Odd";
    }
}
```
