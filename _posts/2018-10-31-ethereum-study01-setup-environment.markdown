---
layout: post
title:  "이더리움 스터디 #1 환경 구성"
author: 윤상준
date: 2018-10-31
categories: ethereum
tags:
- ethereum
- geth
- smartcontract
- dapp
- mining
---

이 페이지에서는 이더리움 환경에서 Smart Contract와 DApp을 개발하기 위한 환경을 구성하는 방법을 알아보겠습니다.

## Geth 설치

먼저 이더리움 API를 사용하기 위한 Geth를 설치하는 방법에 대해 알아보겠습니다.

아래 Github 저장소에서 Geth 소스코드를 복사합니다.

```
git clone https://github.com/ethereum/go-ethereum.git

cd go-ethereum
```

Stable 버전을 checkout 합니다.

```
git checkout release/1.7
```

Geth를 컴파일합니다.

```
$ make geth
...
Done building.
Run "PATH/go-ethereum/build/bin/geth" to launch geth.
```

Geth binary를 `/usr/local/bin/`으로 복사합니다.

```
cp PATH/go-ethereum/build/bin/geth /usr/local/bin/
```

Geth 버전을 확인합니다.

```
$ geth version
Geth
Version: 1.7.3-stable
Git Commit: 4bb3c89d44e372e6a9ab85a8be0c9345265c763a
Architecture: amd64
Protocol Versions: [63 62]
Network Id: 1
Go Version: go1.9.3
Operating System: darwin
GOPATH=PATH/go
GOROOT=/usr/local/go
```

* Geth 컴파일 중 아래와 같은 에러가 발생하면 Go version을 `1.9.3`으로 재설치합니다.

```
$ make geth
build/env.sh go run build/ci.go install ./cmd/geth
ci.go:183: You have Go version go1.x.x
ci.go:184: go-ethereum requires at least Go version 1.7 and cannot
ci.go:185: be compiled with an earlier version. Please upgrade your Go installation.
exit status 1
make: *** [geth] Error 1
```

## Geth 환경 초기화

`genesis.json` 파일을 생성합니다.

```
{
    "config": {
        "chainId": 1,
        "homesteadBlock": 0,
        "eip155Block": 0,
        "eip158Block": 0
    },
    "coinbase" : "0x0000000000000000000000000000000000000000",
    "difficulty" : "0x05",
    "extraData" : "0x00",
    "gasLimit" : "0x80000000",
    "nonce" : "0x0000000000000000",
    "mixhash" : "0x0000000000000000000000000000000000000000000000000000000000000000",
    "parentHash" : "0x0000000000000000000000000000000000000000000000000000000000000000",
    "timestamp" : "0x00",
    "alloc" : {
    }
}
```

datadir로 지정할 노드의 폴더를 생성합니다.
이 페이지에서는 node1으로 생성하겠습니다.

```
mkdir node1
```

geth 명령을 실행하여 초기화 합니다.

```
geth --datadir node1 init genesis.json
```

## Geth console 실행

Ethereum main net이 아닌 Private network를 사용해서 Geth console을 실행합니다.

```
$ geth --networkid 8484 --nodiscover --maxpeers 0 --datadir node1 console 2>> node1/geth.log
Welcome to the Geth JavaScript console!

instance: Geth/v1.7.3-stable-4bb3c89d/darwin-amd64/go1.9.3
 modules: admin:1.0 debug:1.0 eth:1.0 miner:1.0 net:1.0 personal:1.0 rpc:1.0 txpool:1.0 web3:1.0

>
```

아래는 geth 명령의 각 옵션의 의미입니다.

더 자세한 사항은 Geth Github 페이지를 참고하세요.
