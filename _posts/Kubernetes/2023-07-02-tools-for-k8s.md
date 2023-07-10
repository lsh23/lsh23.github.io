---
title: kubernetes 유용한 툴
date: 2023-07-02 16:08:16 +0900
categories: [ Kubernetes ]
tags: [ Kubernetes ]
comments: true
---
kubernetes를 사용하는데 있어어 유용한 툴에 대한 간단 소개 및 설치 방법

## 개요
kubernetes 클러스터에 접근하고 명령을 내리기위해서 기본적으로 사용하는 CLI 도구는 kubectl이다.  
이번 글에서는 kubernetes를 사용할 때 유용한 CLI 툴에 대해 간단히 소개하며, mac os 기준으로 설치 방법에 대해서도 소개한다.

## 자동완성
kubectl CLI는 기본적으로 자동완성이 제공되지 않는다. 따라서 특정 pod의 정보를 세부 정보를 조회하고 싶을 때 `kubectl get pod` 명령어를 통해서 pod이름을 조회하고 복사한뒤 `kubectl describe pod pod명` 명령을 통해서 조회를 하는데 이게 생각보다 귀찮다. 아래의 자동완성 설정으로 resource명 뿐만 아니라 command도 자동완성이 되서 매우 편리하게 kubectl CLI를 사용할 수 있다.

### 설정 방법(zsh)
```
source <(kubectl completion zsh)
```
{: file='~/.zshrc'}
.zshrc 파일에 위의 명령을 추가해주면 된다.  
`source ~/.zshrc`명령으로 업데이트된 .zshrc 파일을 현재 shell session에 반영해주거나 shell을 새로 띄우면 kubectl 명령과 tab키를 조합해서 자동완성을 사용할 수 있다.
만약에 `2: command not found: compdef` 에러문구가 발생하면서 자동완성이 되지 않는 경우에는 
```
autoload -Uz compinit
compinit
...
...
```
{: file='~/.zshrc'}
위의 2줄을 .zshrc 파일 맨위에 추가하고 다시 .zshrc 파일을 현재 shell session에 반영해주고 동작을 확인해보자.

## kubectx & kubens 

여러개의 kubernetes 클러스터를 관리하는 경우 kubectl로 클러스터를 전환하려면 `kubectl config use-context $클러스터이름` 으로 전환하거나, 
같은 클러스터내에서 default 네임스페이스가 아닌 특정네임스페이스의 resourece를 조회할떄 옵션에 `-n ` 또는 `--namesapce=` 를 사용해야하는데 막상 사용해보면 은근 귀찮고 거슬린다.  
쉽게 클러스터와 default 네임스페이스를 전환할 수 있는 kubectx, kubens를 소개한다.

### 설치방법
```shell
$ brew install kubectx
```
kubectx가 설치되면서 kubens도 같이 설치된다.

### 사용방법
```bash
$ kubectx my-cluster-1 # 클러스터명이 my-cluster-1인 클러스터로 전환
$ kubens argocd       # 네임스페이스명이 argocd인 네임스페이스를 default로 지정
``` 

## kubectl neat
개인적으로 학습용도로 좋다고 생각하는 도구다.  
kubernetes에 helm charts로 argocd나 elastic search와 같은 도구들을 쉽게 설치할 수 있다. 이 때 해당 helm chart로 배포된 pod의 manifest 정보를 학습차원에서 참고해보고 싶을 떄가 있었다.  
직접 공식 helm chart의 repo를 통해서 확인을 할 수 있지만 kubectl 명령어로도 충분히 확인할 수 있다.  
하지만 kubectl get pod -o yaml 명렁어로 pod에 대한 manifest 정보를 조회할 수 있지만 해당 명령으로 조회를 하면 불필요한 정보(내부 시스템 정보, 기본 디폴트 값)까지 엄청나게 장황하게 조회가 된다.  
이 때 kubectl neat를 사용하면 실제 manifest file(.yaml)을 작성할 때 넣어주는 값들만 추려서 보여줘서 manifest를 어떻게 구성하면 좋을지에 대한 공부나 힌트를 얻을 수 있다.

### 설치방법 
```shell
$ brew install git
$ set -x; cd "$(mktemp -d)" &&
$ OS="$(uname | tr '[:upper:]' '[:lower:]')" &&
$ ARCH="$(uname -m | sed -e 's/x86_64/amd64/' -e 's/\(arm\)\(64\)\?.*/\1\2/' -e 's/aarch64$/arm64/')" &&
$ KREW="krew-${OS}_${ARCH}" &&
$ curl -fsSLO "https://github.com/kubernetes-sigs/krew/releases/latest/download/${KREW}.tar.gz" &&
$ tar zxvf "${KREW}.tar.gz" &&
$ ./"${KREW}" install krew
$ echo 'export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"' >> ~/.zshrc

$ source ~/.zshrc
$ kubectl krew version # krew 설치확인
$ kubectl krew install neat # krew를 통해 neat 설치
```
kubectl neat는 kubectl에 neat plugin을 설치하는 방식이다.
kubectl에 plugin을 설치하기위해 kubectl plugin manager인 krew를 설치하고, krew를 통해서 neat를 설치한다.

### 사용방법

```shell 
$ kubectl get pod mypod -o yaml           # 기존 조회 
$ kubectl get pod mypod -o yaml | k neat  # k neat로 필터링
```
`|` 파이프를 통해서 k neat의 input으로 필터링되기전 값을 넘겨주면된다. 글의 가독성을 위해서 조회 결과는 첨부하지 않았다. 직접 설치 후 사용해보고 필터링 전후의 결과를 비교해보자.


## Refercence
* <https://kubernetes.io/ko/docs/tasks/tools/included/optional-kubectl-configs-zsh/>
* <https://kubernetes.io/ko/docs/tasks/tools/included/optional-kubectl-configs-bash-linux/>
* <https://github.com/ahmetb/kubectx/>
* <https://github.com/itaysk/kubectl-neat/>