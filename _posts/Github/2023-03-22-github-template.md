---
title: Gihub issue, pr template 등록하기
date: 2023-03-22 11:10:16 +0900
categories: [ Github ]
tags: [ Github ]
comments: true
---

Github에서 issue, pull request template 등록하기
## 개요
오픈소스소 내 소스코드를 github에 공개하거나, 여러명이서 github를 이용해 프로젝트를 진행하다보면,  
여러사람이 issue를 등록한다거나, PR을 요청하게 된다. 이때 특정한 양식이 없다면 서로 다른 포맷으로 작성이 되므로 일관성있게 관리하기가 어렵다.  
사전에 template을 등록해서 통일성 있게 issue, PR이 요청될 수 있도록 하는 방법에 대해서 알아보자.

## Issue template 설정 
### 1. template 설정 메뉴 들어가기
git repository settings -> Features의 issues 탭에서 Set up templates 클릭
![Desktop View](/posts/2023-03-22-github-template/1.png)
![Desktop View](/posts/2023-03-22-github-template/2.png)
### 2. template 편집하기  
Add template에서 기본 template 선택 후    
template 이름, 설명, 본문, default title, assignee, label 등을 설정해준다.
![Desktop View](/posts/2023-03-22-github-template/3.png)
![Desktop View](/posts/2023-03-22-github-template/4.png)
### 3. template 등록
Propose changes 버튼을 이용해서 template file을 commit 해준다.  
![Desktop View](/posts/2023-03-22-github-template/5.png)
`${repo_root}/.github/ISSUE_TEMPLATE/`{: .filepath} 경로에 template_name.md 형태로 template 파일이 저장된다.
![Desktop View](/posts/2023-03-22-github-template/8.png)

### 4. template 확인
git repository issues 탭을 들어가보면 등록한 template으로 issue 등록하기가 활성화 된다.  
등록한 template을 기반으로 issue 작성이 가능하다.
![Desktop View](/posts/2023-03-22-github-template/6.png)
![Desktop View](/posts/2023-03-22-github-template/7.png)


## PR template 설정 
### 1. template file 작성 
파일 이름을 pull_request_template.md으로 마크다운으로 template을 작성하고  
`${repo_root}/.github/pull_request_template.md`{: .filepath} 경로로 해당 파일을 commit / push 해준다.
```md
## 구현 기능
- 구현한 기능

## 학습한 내용
- 학습한 내용

## 기타
- 없는 경우 삭제

## Close
- close #issue
```
{: file='pull_request_template.md'}

### 2. template 확인
등록한 template을 기반으로 PR 작성이 가능하다.
![Desktop View](/posts/2023-03-22-github-template/9.png)


## Refercence
* <https://docs.github.com/en/communities/using-templates-to-encourage-useful-issues-and-pull-requests/configuring-issue-templates-for-your-repository>
* <https://docs.github.com/en/communities/using-templates-to-encourage-useful-issues-and-pull-requests/creating-a-pull-request-template-for-your-repository>
