---
title: submodule로 민감정보 관리하기
date: 2023-06-08 16:26:13 +0900
categories: [ Spring ]
tags: [ Spring, Git, Submodule ]
comments: true
---
Submodule로 민감정보 관리하기
## 개요
웹 애플리케이션을 개발하다보면 OAUTH Client, Secret Key, JWT Secret Key와 같은 민감정보에 대한 소스코드를 어떻게 관리해야할지 고민해야하는 시점이온다.  
그냥 commit을 찍고 공개된 저장소에 소스코드를 업로드를 하게 되면 민감정보들이 외부에 노출되기 때문이다.  
이번 글에서는 git submodule로 민감정보를 private repo를 분리해서 다루는 방법에 대해서 알아본다.

## submodule 이란
submodule은 하나의 repository 안에 또 다른 별개의 repository 저장소를 관리할 수 있는 기능이다.  

## submodule 준비하기
submodule 준비는 특별한 것이 없다. 그냥 새로운 git private repository를 만들고 민감정보를 담고있는 설정파일이나 소스코드를 commit하고 push하면 된다.

## 메인 repository에 submodule 추가하기
메인 repository에서 submodule을 추가할 경로로 들어가서 submodule을 아래의 명령어로 추가한다.
```console
$ cd ~/demo-project/src/main/resources
$ git submodule add https://github.com/lsh23/demo-project-config.git config
```
~/demo-project/src/main/resources/config 폴더가 생성되고 소스코드를 당겨오는 것을 확인할 수 있다.


## 메인 repository를 clone할 때
submodule을 포함하고 있는 repository의 소스코드를 받아야할 때는 아래의 명령어로 받아 올 수 있다. 

```console
$ git clone ${main_repo_clone_url}
$ cd ${main_repo_folder}

$ git submodule init
$ git submodule update
```

## 부록 - git action에서 submodule pull 추가하기
git action이 아니더라도 자동 CI를 구성해놓았더라면 소스코드를 pull할때 submodule 역시 같이 pull하도록 설정을 추가해줘야한다.  
git action을 사용한다면 아래와 같은 설정을 추가해서 간단하게 해결할 수 있다.  

{% raw %}
```diff
    steps:
    - uses: actions/checkout@v3
+      with:
+        token: ${{ secrets.TOKEN_FOR_PULLING }}
+        submodules: true
    - name: Set up JDK 17
      uses: actions/setup-java@v3
      with:
```
{% endraw %}

submodule: true와 submoudle repo에 권한이 있는 github token을 지정해주면된다.  

## Refercence
* <https://git-scm.com/book/en/v2/Git-Tools-Submodules/>
* <https://github.com/marketplace/actions/checkout/>
* <https://github.com/marketplace/actions/checkout-submodules?version=2.1.1/>
