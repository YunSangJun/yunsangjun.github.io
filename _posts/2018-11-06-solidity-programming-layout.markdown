---
layout: post
title:  "Solidity Programming #2 소스코드 레이아웃"
author: 윤상준
date: 2018-11-06
categories: solidity
tags:
- ethereum
- solidity
- smartcontract
---

이 페이지에서는 Solidity 소스코드의 레이아웃에 대해 살펴보겠습니다.

## Version Pragma

소스코드가 호환되지 않는 버전으로 컴파일되는 것을 방지하기 위해 Version Pragma를 사용할 수 있습니다.

아래와 같이 컴파일러의 버전을 선언해 특정 버전으로 컴파일 되도록 설정할 수 있습니다.

```
pragma solidity ^0.x.x;
```

## File import

- 동일 환경의 file import

  ```
  import "./filename.sol";
  ```

- 원격 경로의 file import

  ```
  import "github.com/xxx/filename.sol";
  ```

## Comments

- Single line

  ```
  // This is a single-line comment.
  ```

- Multi line

  ```
  /*
  This is a
  multi-line comment.
  */
  ```

- Documentation

  ```
  pragma solidity ^0.x.x;

  /** @title Shape calculator. */
  contract shapeCalculator {
      /** @dev Calculates a rectangle's surface and perimeter.
        * @param w Width of the rectangle.
        * @param h Height of the rectangle.
        * @return s The calculated surface.
        * @return p The calculated perimeter.
        */
      function rectangle(uint w, uint h) returns (uint s, uint p) {
          s = w * h;
          p = 2 * (w + h);
      }
  }
  ```
