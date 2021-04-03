---
layout: post
title:  "이더리움 스터디 #4 JSON RPC(remote procedure call) API"
author: sj
date: 2018-11-02
categories: ethereum
tags:
- ethereum
- geth
- smartcontract
- dapp
- mining
---

이 페이지에서는 JSON RPC(remote procedure call) API를 사용하여 ethereum 기능을 사용하는 방법에 대해 알아보겠습니다.

## JSON RPC(remote procedure call) API

JSON RPC API를 사용하여 잔고를 조회하는 등의 ethereum 기능을 사용할 수 있습니다.

API에 대한 상세한 정보는 [JSON RPC API](https://github.com/ethereum/wiki/wiki/JSON-RPC)를 참고하세요.

### Geth console 실행

아래와 같이 Geth console을 rpc 모드로 실행합니다.

rpc port를 `8545`로 실행합니다.

```
$ geth --networkid 8484 --nodiscover --maxpeers 0 --rpc --rpcaddr "0.0.0.0" --rpcport 8545 --rpccorsdomain "*" --datadir node1 console 2>> node1/geth.log
```

### Client 버전 확인

Geth client 버전을 확인해보겠습니다.

curl 명령을 사용해 localhost의 `8545` port로 요청을 전송합니다.

Header, data등의 함께 넘겨야 하는 값은 Content type은 [JSON RPC API](https://github.com/ethereum/wiki/wiki/JSON-RPC)를 참고하여 작성합니다.

API 문서에서 Client version을 확인하는 `web3_clientVersion` method에 대한 정보를 확인해서 작성합니다.

```
$ curl http://localhost:8545 -H "Content-Type:application/json" -X POST --data '{"jsonrpc":"2.0","method":"web3_clientVersion", "params":[], "id":1}'
{"jsonrpc":"2.0","id":1,"result":"Geth/v1.7.3-stable-4bb3c89d/darwin-amd64/go1.9.3"}
```

응답 메세지에 result의 값에서 Geth client 버전을 확인할 수 있습니다.

### 잔고(Balance) 확인

이번에는 잔고를 확인해보겟습니다.

API 문서에서 잔고를 확인하는 `eth_getBalance` method에 대한 정보를 확인해서 작성합니다.

```
$ curl http://localhost:8545 -H "Content-Type:application/json" -X POST --data '{"jsonrpc":"2.0","method":"eth_getBalance", "params":["0x455def09982bce61ddd29e833a79731d7fab7fac", "latest"], "id":1}'
{"jsonrpc":"2.0","id":1,"result":"0x8670e9ec6598c0000"}
```

응답 메세지에 result 값에서 잔고를 확인할 수 있습니다.

`0x8670e9ec6598c0000`는 16진수 값으로 10진수로 변환하면 `155000000000000000000` 즉 '155' ether 인것을 확인할 수 있습니다.
