---
layout: post
title:  "Docker Private Registry로 부터 이미지 가져오기"
author: sj
date: 2018-05-25
categories: kubernetes
tags:
- kubernetes
- container
- docker
- image
- secret
---

이 페이지는 Secret을 사용하여 Private Docker Registry에서 이미지를 가져오는 Pod를 만드는 방법을 보여줍니다.<br>
상세한 내용은 아래 링크를 참고하세요.

https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/

## Docker Private Registry에 로그인

개인 이미지를 가져오려면 Registry로 인증해야합니다.
```
docker login YOUR_REPOSITORY_URI
```

`config.json` 파일을 조회합니다. 해당 Repository에 대한 인증 토큰 정보가 출력됩니다.
```
$ cat ~/.docker/config.json
{
    "auths": {
        "YOUR_REPOSITORY_URI": {
            "auth": "c3R...zE2"
        }
    }
}
```

## 인증 토큰을 사용하여 Secret 생성하기

Kubernetes 클러스터는 docker-registry 유형의 Secret을 사용하여 컨테이너 레지스트리로 인증하여 개인 이미지를 가져옵니다.

이 Secret 만들어서 이름을 regcred로 지정하십시오.

```
kubectl create secret docker-registry regcred --docker-server=<your-registry-server> --docker-username=<your-name> --docker-password=<your-pword> --docker-email=<your-email>
```
- your-registry-server : Private Docker Registry FQDN.
- your-name : Docker username.
- your-pword : Docker password.
- your-email : Docker email.

## Secret 확인하기

`regcred` Secret 내용 조회

```
$ kubectl get secret regcred --output=yaml
apiVersion: v1
data:
  .dockercfg: eyJodHRwczovL2luZGV4L ... J0QUl6RTIifX0=
kind: Secret
metadata:
  ...
  name: regcred
  ...
type: kubernetes.io/dockercfg
```

`.dockercfg` 정보 Decode

```
$ kubectl get secret regcred --output="jsonpath={.data.\.dockercfg}" | base64 --decode
{"auths":{"yourprivateregistry.com":{"username":"janedoe","password":"xxxxxxxxxxx","email":"jdoe@example.com","auth":"c3R...zE2"}}}
```

인증 토큰 조회

```
$ echo "c3R...zE2" | base64 -d
janedoe:xxxxxxxxxxx
```

## Secret을 사용하여 Pod 생성하기

Pod yaml 생성

```
$ vi my-private-reg-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: private-reg
spec:
  containers:
  - name: private-reg-container
    image: <your-private-image>
  imagePullSecrets:
  - name: regcred
```

Pod 배포. 정상적으로 완료했다면 이미지를 Docker Private Registry로 부터 가져오고 Pod가 Running 상태로 변경됩니다.

```
$ kubectl create -f my-private-reg-pod.yaml
```
