---
layout: post
title:  "Virtual Kubelet을 활용해 Kubernetes를 서버리스 컨테이너로 확장하기"
author: sj
date: 2019-08-25
categories: kubernetes
tags:
- kubernetes
- virtualkubelet
- cloud
- container
---

## Overview
Private 환경에서 kubernetes cluster를 운영하고 있다면 workload를 bursting 하는 관점에서 
Public cloud와 연계하는 방안을 한번쯤은 고민해봤을 것이라 생각합니다.

이와 관련해 CNCF(Cloud Native Computing Foundation)에 
[Virtual Kubelet](https://github.com/virtual-kubelet/virtual-kubelet)이라는
흥미로운 프로젝트가 있어 활용 사례 및 Hands-on을 남겨봅니다.

Virtual Kubelet에 대한 설명을 보면 `Kubernete를 다른 API와 연동하기 위한 Kubelet 구현체`라고 정의되어 있습니다.
이 설명만 들으면 활용 방법이나 프로젝트에서 추구하는 목적이 크게 와닿지가 않습니다.

이해를 돕기 위해 활용 사례를 들어보겠습니다.
아래 그림과 같이 운영중인 Kubernetes cluster에 Public Cloud의 서버리스 컨테이너 서비스
(예 - AWS ECS Fargate, Azure Container Instance 등)를 확장해 마치 하나의 cluster 처럼 사용할 수 있습니다.  

![](/assets/images/kubernetes/virtual-kubelet/virtual-kubelet-architecture.png)

이를 통해 컨테이너 서비스 별로 서로 다른 사용 방식을 고려할 필요 없이 Kubernetes stack으로 통일하여 개발 및 운영을 할 수 있는 장점이 있습니다.
또한 Public cloud에는 Kubernetes cluster를 추가로 운영할 필요없이 컨테이너에 대한 비용만 지불하면 되므로 비용 효율적인 측면이 있습니다.

물론 제약 사항도 있습니다. 아래 현재 지원하는 feature를 보면 kubernetes의 모든 기능을 사용할 수 있는 것은 아닙니다.
- create, delete and update pods
- container logs, exec, and metrics
- get pod, pods and pod status
- capacity
- node addresses, node capacity, node daemon endpoints
- operating system
- bring your own virtual network

또한 Virtual Kubelet에서 사용할 virtual network를 지정할 수 있지만 
해당 network가 Kubernetes cluster의 pod에서 사용하는 network과 연결성이 없다면 
pod간 private 통신도 불가능합니다.

추가로 프로젝트 설명에 Multi Kubernetes cluster를 federation 하기 위한 용도가 아니라고 명시되어 있으니
사용에 참고가 필요할 거 같습니다.

마지막으로 프로젝트 현황을 살펴보면 CNCF의 Sandbox 프로젝트라고 합니다.
일반적으로 Sandbox => Incubation => Graduate 순으로 발전된다고 합니다.

최근 1.0 버전이 릴리스되어 프로젝트가 안정화되어 가는거 같고 
실용적인 활용 사례와 컨셉이 명확해 앞으로 기대가 되는 프로젝트입니다.

아래 hands-on을 한번 따라해보시면 프로젝트 컨셉과 활용 방법에 대해 좀 더 쉽게 이해할 수 있으리라 생각됩니다.

## 사전 준비하기

이 문서에서는 누구나 따라할 수 있도록 Public cloud의 무료 체험이 가능한 환경 기준으로 작성했습니다.

Kubernetes cluster는 GKE(Google Kubernetes Engine), 컨테이너 서비스는 ACI(Azure Container Instance)를 사용했습니다.

Hands-on을 위해서 아래와 같은 준비사항이 필요합니다.

1. Kubernetes cluster

    기존의 보유하고 있는 cluster를 사용하거나 새로 생성합니다.

    [GKE(Google Kubernetes Engine)](https://cloud.google.com/kubernetes-engine/docs/quickstart?hl=ko),
    [AKS(Azure Kubernetes Service)](https://azure.microsoft.com/ko-kr/services/kubernetes-service/), 
    [EKS(Elastic Kubernetes Service)](https://aws.amazon.com/ko/eks/) 
    등 관리형 Kubernetes 서비스를 활용하여 쉽고 빠르게 생성할 수 있습니다.

    이 문서에서는 GKE 환경을 사용했습니다. 구글 계정을 가지고 있다면 Credit을 받아 일정기간 무료로 사용할 수 있습니다.

2. Azure 계정

    ACI(Azure Container Instance)를 사용하기 위해 Azure 계정을 준비합니다.

    Azure 계정이 없다면 [Azure 체험 계정 만들기](https://azure.microsoft.com/ko-kr/free/)를 통하여 
    계정을 생성하고 Credit을 받아 일정기간 무료로 사용할 수 있습니다.

3. Azure CLI.

    Azure CLI를 설치합니다.

    - OSX 

        ```
        brew install azure-cli
        ```

    - Linux(Ubuntu 64-bit)

        ```
        $ echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ wheezy main" | \
        sudo tee /etc/apt/sources.list.d/azure-cli.list
 
        $ sudo apt-key adv --keyserver packages.microsoft.com --recv-keys 52E16F86FEE04B979B07E28DB02C46DF417A0893
        $ sudo apt-get install apt-transport-https
        $ sudo apt-get update && sudo apt-get install azure-cli
        ```

4. Kubernetes CLI

    [Kubernetes CLI](https://kubernetes.io/docs/tasks/tools/install-kubectl/)를 설치합니다.

5. Helm CLI.

    [Helm CLI](/blog/helm/2018/05/27/installing-helm.html)를 설치합니다.

## Azure 계정 설정하기

1. Azure에 로그인합니다.

    ```
    az login
    ```

2. Azure subscription 조회 및 복사합니다.

    ```
    $ az account list -o table
    Name       CloudName    SubscriptionId                        State    IsDefault
    ---------  -----------  ------------------------------------  -------  -----------
    무료 체험    AzureCloud   <SubscriptionId>                      Enabled  True
    ```

3. Subscription ID에 위에서 복사한 ID를 저장합니다.

    ```
    export AZURE_SUBSCRIPTION_ID="<SubscriptionId>"
    ```

4. ACI(Azure Container Instance)를 활성화합니다.

    ```
    az provider register -n Microsoft.ContainerInstance
    ```

## ACI Resource Group 생성하기

<p class="tip-title">참고</p>
<p class="tip-content">
현재 Azure 한국 리전에서는 ACI 서비스가 없습니다. 
서비스 중인 가장 가까운 리전은 japaneast 입니다.(japanwest도 없음)
</p>

```
export ACI_REGION=japaneast
az group create --name aci-group --location "$ACI_REGION"
export AZURE_RG=aci-group
```

## Azure Service principal 생성하기

1. RBAC과 함께 SP(Service principal)를 생성합니다.

    ```
    $ az ad sp create-for-rbac --name virtual-kubelet-quickstart -o table
    ...
    AppId       DisplayName        Name                Password                  Tenant
    -----------------------------------------------------------------------------------------------  
    <AppId>     ...                ...                 <Password>                <Tenant>
    ```

2. 위의 결과를 변수로 저장합니다. 

    ```
    export AZURE_TENANT_ID=<Tenant>
    export AZURE_CLIENT_ID=<AppId>
    export AZURE_CLIENT_SECRET=<Password>
    ```

## Virtual Kubelet 배포하기

1. Cluster의 Master Endpoint 확인 및 복사합니다.

    ```
    $ kubectl cluster-info
    Kubernetes master is running at <Kubernetes Master>
    ...

    $ export MASTER_URI=<Kubernetes Master>
    ```

2. Helm을 통하여 Virtual Kubelet 설치합니다.

    ```
    VK_RELEASE=virtual-kubelet-latest
    RELEASE_NAME=virtual-kubelet
    NODE_NAME=virtual-kubelet
    CHART_URL=https://github.com/virtual-kubelet/virtual-kubelet/raw/master/charts/$VK_RELEASE.tgz

    helm install "$CHART_URL" --name "$RELEASE_NAME" \
    --set provider=azure \
    --set rbac.install=true \
    --set providers.azure.targetAKS=false \
    --set providers.azure.aciResourceGroup=$AZURE_RG \
    --set providers.azure.aciRegion=$ACI_REGION \
    --set providers.azure.tenantId=$AZURE_TENANT_ID \
    --set providers.azure.subscriptionId=$AZURE_SUBSCRIPTION_ID \
    --set providers.azure.clientId=$AZURE_CLIENT_ID \
    --set providers.azure.clientKey=$AZURE_CLIENT_SECRET \
    --set providers.azure.masterUri=$MASTER_URI
    ```

3. 정상 설치 확인하기

    virtual-kubelet이 cluster에 Pod로 배포되었고, Running 중인지 확인합니다.

    ```
    $ kubectl --namespace=default get pods -l "app=virtual-kubelet"
    NAME                                               READY   STATUS    RESTARTS   AGE
    virtual-kubelet-virtual-kubelet-7c9fbc55c4-r2j57   1/1     Running   0          23s
    ```

    Cluster의 node를 조회해 virtual-kubelet node가 조회되는지 확인합니다.

    ```
    $ kubectl get nodes
    NAME                                                STATUS   ROLES    AGE   VERSION
    gke-standard-cluster-1-default-pool-cc858c9e-9sf8   Ready    <none>   39m   v1.12.8-gke.10
    gke-standard-cluster-1-default-pool-cc858c9e-bm01   Ready    <none>   39m   v1.12.8-gke.10
    virtual-kubelet                                     Ready    agent    44s   v1.13.1-vk-v0.9.0-1-g7b92d1ee-dev
    ```

## Sample App 배포하기

1. virtual-node.yaml 파일을 생성합니다.
ACI(Azure Container Instance)에 배포하기 위해 nodeSelector와 toleration을 추가합니다.

    ```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: aci-helloworld
spec:
  replicas: 1
  selector:
    matchLabels:
      app: aci-helloworld
  template:
    metadata:
      labels:
        app: aci-helloworld
    spec:
      containers:
      - name: aci-helloworld
        image: microsoft/aci-helloworld
        ports:
        - containerPort: 80
      nodeSelector:
        kubernetes.io/role: agent
        beta.kubernetes.io/os: linux
        type: virtual-kubelet
      tolerations:
      - key: virtual-kubelet.io/provider
        operator: Exists
      - key: azure.com/aci
        effect: NoSchedule
    ```

2. aci-helloworld pod를 배포합니다.

    ```
    $ kubectl apply -f virtual-node.yaml
    ```

3. 배포 후 Pod를 조회해보면 aci-helloworld app은 virtual-kubelt node에 배포된 것을 확인할 수 있습니다.

    ```
    $ kubectl get po -o wide
    NAME                 READY STATUS    RESTARTS   AGE    IP              NODE              NOMINATED NODE
    aci-helloworld-...   1/1   Running   0          54s    [IP_ADDRESS]    virtual-kubelet   <none>
    sample-app-...       1/1   Running   0          4m17s  10.0.0.7        gke-standard...   <none>
    ```

    aci-helloworld pod의 IP를 보면 azure의 public ip를 할당받은것을 확인할 수 있습니다.
    Virtual Kubelet 배포 시 별도의 vitual network 설정을 하지 않았기 때문입니다.

    aci-helloworld pod의 [IP_ADDRESS]를 정보를 복사한 후 웹브라우저에서 접속하면 
    `Welcome to Azure Container Instances!` 메세지를 확인할 수 있습니다.

    마지막으로 Azure Port에 접속해 ACI resource group에서 aci-helloworld 앱이 배포된것을 확인할 수 있습니다.

    ![](/assets/images/kubernetes/virtual-kubelet/virtual-kubelet-aci.png)

