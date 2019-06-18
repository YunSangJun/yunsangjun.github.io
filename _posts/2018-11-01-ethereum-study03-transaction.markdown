---
layout: post
title:  "이더리움 스터디 #3 송금(Transaction)"
author: 윤상준
date: 2018-11-01
categories: ethereum
tags:
- ethereum
- geth
- smartcontract
- dapp
- mining
---

이 페이지에서는 계정간 송금하는 방법에 대해 알아보겠습니다.

## 계정 생성하기

먼저 송금을 하기 위해 추가로 계정을 생성하겠습니다.

Geth console에서 `user2` 계정을 생성합니다.

```
> personal.newAccount("user2")
"0x9d4820973a161f0d084689ec2f4c3baffc9fd065"
```

계정을 조회합니다.
`0x9d482...`로 시작하는 계정이 생성되었습니다.

```
> eth.accounts
["0x455def09982bce61ddd29e833a79731d7fab7fac", "0x9d4820973a161f0d084689ec2f4c3baffc9fd065"]
```

## 잔고(Balance) 조회

두 개의 계정의 잔고를 조회합니다.
먼저 첫 번째 계정의 잔고를 조회합니다.

```
> web3.fromWei(eth.getBalance(eth.accounts[0]), "ether")
95
```

두 번째 계정의 잔고를 조회합니다.

```
> web3.fromWei(eth.getBalance(eth.accounts[1]), "ether")
0
```

## 송금하기(Transaction)

첫 번째 계정에만 잔고가 있기 때문에 첫 번째에서 두 번째 계정으로 10 ether를 송금해보겠습니다.

```
> eth.sendTransaction({from: eth.accounts[0], to: eth.accounts[1], value: web3.toWei(10, "ether")})
Error: authentication needed: password or unlock
    at web3.js:3143:20
    at web3.js:6347:15
    at web3.js:5081:36
    at <anonymous>:1:1
```

계정이 잠겨있기 때문에 인증 에러가 발생합니다.
첫 번째 계정의 잠금을 해제해보겠습니다. `user1`의 경우 초기 패스워드가 `user1`로 설정되어있습니다.

```
> personal.unlockAccount(eth.accounts[0]);
Unlock account 0x455def09982bce61ddd29e833a79731d7fab7fac
Passphrase:
true
```

다시 송금을 해보겠습니다.

```
> eth.sendTransaction({from: eth.accounts[0], to: eth.accounts[1], value: web3.toWei(10, "ether")})
"0x3d9629b28bf5da3d3d79138edc416b7502165217dd1abd14a73499f125bf0551"
```

`0x3d9629...`로 시작하는 transaction id가 발급됩니다.

또 geth 로그를 조회해보면 해당 transaction id의 로그를 확인할 수 있습니다.
이 로그의 recipient는 ether를 받을 두 번째 계정입니다.

```
$ tail -f node1/geth.log
...
INFO [11-01|17:42:01] Submitted transaction                    fullhash=0x3d9629b28bf5da3d3d79138edc416b7502165217dd1abd14a73499f125bf0551 recipient=0x9d4820973A161F0d084689ec2F4c3BAFFc9fD065
INFO [11-01|17:57:11] Regenerated local transaction journal    transactions=1 accounts=1
...
```

ether를 송금받은 두 번째 계정의 잔고를 확인해보겠습니다.

```
> web3.fromWei(eth.getBalance(eth.accounts[1]), "ether")
0
```

송금은 정상 처리했지만 잔고가 여전히 0입니다.

pending 되어있는 transaction이 있는지 확인해보겠습니다.

```
> eth.pendingTransactions
[{
    blockHash: null,
    blockNumber: null,
    from: "0x455def09982bce61ddd29e833a79731d7fab7fac",
    gas: 90000,
    gasPrice: 18000000000,
    hash: "0x3d9629b28bf5da3d3d79138edc416b7502165217dd1abd14a73499f125bf0551",
    input: "0x",
    nonce: 0,
    r: "0x495c66874abf6d513e47ebf9a2127712903b5028dc0f0bcb75fe2baf0ab57eed",
    s: "0x20d9c240ec64c7395d2f33dc897fb5022011fe57986210892af7b4ae3ba1a6d3",
    to: "0x9d4820973a161f0d084689ec2f4c3baffc9fd065",
    transactionIndex: 0,
    v: "0x26",
    value: 10000000000000000000
}]
```

`0x3d9629...`로 시작하는 transaction이 pending 되어 있는 것을 확인할 수 있습니다.
아직 블록에 담기지 못한 transaction은 노드의 transaction pool에 저장됩니다.

여기서 `gas`와 `gasPrice`라는 것을 확인할 수 있는데 채굴에 대한 보상으로 제공하는 수단입니다.
이 pending되어 있는 transaction을 처리하기 위해서는 채굴을 해서 gas를 보상으로 제공해야합니다.

아래와 같이 채굴을 시작합니다.

```
> miner.start()
null
```

pending 되어있는 transaction이 있는지 확인해보겠습니다.

```
> eth.pendingTransactions
[]
```

pending 되어있는 transaction이 없으면 송금이 완료된것입니다.

송금이 완료되었기 때문에 채굴을 중지합니다.

```
> miner.stop()
true
```

이제 다시 두 번째 계정의 잔고를 확인해보겠습니다.

```
> web3.fromWei(eth.getBalance(eth.accounts[1]), "ether")
10
```

10 ether 송금이 완료된 것을 확인할 수 있습니다.
