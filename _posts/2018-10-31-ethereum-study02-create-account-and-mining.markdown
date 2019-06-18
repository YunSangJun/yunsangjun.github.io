---
layout: post
title:  "이더리움 스터디 #2 계정 생성 및 채굴(Mining)"
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

이 페이지에서는 이더리움 환경에서 Smart Contract와 DApp을 개발하기 위한 계정 생성하는 방법을 알아보겠습니다.

## 계정 생성하기

Geth console에서 `user1` 계정을 생성합니다.

```
> personal.newAccount("user1")
"0x0dc9e7c16d3d685ae4bd67c21c7d25cfebaace8d"
```

계정을 조회합니다.

```
> eth.accounts
["0x0dc9e7c16d3d685ae4bd67c21c7d25cfebaace8d"]
```

메인 계정을 조회합니다. 채굴을 하면 보상이 메인 계정에 쌓이게됩니다.
```
> eth.coinbase
"0x0dc9e7c16d3d685ae4bd67c21c7d25cfebaace8d"
```

## 채굴하기(Mining)

계정의 잔고를 확인합니다. 현재는 잔고가 0입니다. 채굴을 해서 잔고가 늘어나는 것을 확인해보겠습니다.

```
> eth.getBalance(eth.coinbase)
0
```

채굴을 하기전에 채굴이 잘 되고 있는지 확인하기 위해 로그를 확인하는 방법을 설명하겠습니다.

앞에서 Geth console을 실행할 때 아래와 같이 로그를 파일로 저장하도록 설정했습니다.
```
geth --networkid 8484 --nodiscover --maxpeers 0 --datadir node1 console 2>> node1/geth.log
Welcome to the Geth JavaScript console!
```

새로운 터미널을 열고 tail 명령으로 해당 로그를 실시간 조회합니다.

```
$ tail -f node1/geth.log
...
INFO [10-31|02:30:49] Disk storage enabled for ethash caches   dir=/Users/sangjunyun/ethereum/node1/geth/ethash count=3
INFO [10-31|02:30:49] Disk storage enabled for ethash DAGs     dir=/Users/sangjunyun/.ethash                    count=2
INFO [10-31|02:30:49] Initialising Ethereum protocol           versions="[63 62]" network=8484
INFO [10-31|02:30:49] Loaded most recent local header          number=0 hash=e56eca…30a326 td=5
INFO [10-31|02:30:49] Loaded most recent local full block      number=0 hash=e56eca…30a326 td=5
INFO [10-31|02:30:49] Loaded most recent local fast block      number=0 hash=e56eca…30a326 td=5
INFO [10-31|02:30:49] Regenerated local transaction journal    transactions=0 accounts=0
INFO [10-31|02:30:49] Starting P2P networking
INFO [10-31|02:30:49] RLPx listener up                         self="enode://84c6667d7d04d32884b8c7439734822b341473e12048e4fd82e8f86b8e98d55a8ece1f35d402ca98faf8edbb0e44005856701cc0bbbee4d7048e208ebd769406@[::]:30303?discport=0"
INFO [10-31|02:30:49] IPC endpoint opened: /Users/sangjunyun/ethereum/node1/geth.ipc
```

다시 Geth console로 돌아와서 채굴을 시작합니다.

```
> miner.start()
null
```

채굴이 시작되면 `geth.log`에 실시간으로 로그가 출력됩니다.

`Generating DAG in progress` 메세지가 일정 시간 출력되다가 `mined potential block` 메세지가 출력되면
채굴이 시작됩니다.

```
$ tail -f node1/geth.log
...
INFO [10-31|02:54:49] Updated mining threads                   threads=0
INFO [10-31|02:54:49] Transaction pool price threshold updated price=18000000000
INFO [10-31|02:54:49] Starting mining operation
INFO [10-31|02:54:49] Commit new mining work                   number=1 txs=0 uncles=0 elapsed=296.949µs
INFO [10-31|02:54:53] Generating DAG in progress               epoch=0 percentage=0 elapsed=2.333s
...
INFO [10-31|02:58:38] Generating DAG in progress               epoch=0 percentage=99 elapsed=3m48.053s
INFO [10-31|03:00:27] Successfully sealed new block            number=1 hash=bae3db…201918
INFO [10-31|03:00:27] 🔨 mined potential block                  number=1 hash=bae3db…201918
INFO [10-31|03:00:27] Commit new mining work                   number=2 txs=0 uncles=0 elapsed=25.996ms
INFO [10-31|03:00:30] Generating DAG in progress               epoch=1 percentage=24 elapsed=1m38.527s
INFO [10-31|03:00:31] Successfully sealed new block            number=2 hash=4db296…447bfe
...
```

Geth console에서 잔고를 다시 조회합니다.
`web3.fromWei` 명령을 사용하면 `ether` 단위로 잔고를 조회할 수 있습니다.

```
> eth.getBalance(eth.accounts[0])
50000000000000000000
> web3.fromWei(eth.getBalance(eth.coinbase), "ether")
50
```

Block이 몇개 생성되었는지 조회해봅니다.

```
> eth.blockNumber
19
```

잔고가 어느정도 쌓였다면 채굴을 중지합니다.

```
> miner.stop()
true
```
