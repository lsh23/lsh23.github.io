---
title: mysql 유저 생성 및 권한 부여
date: 2023-04-20 23:35:23 +0900
categories: [ MySQL ]
tags: [ MySQL, grant ]
comments: true
---
MySQL에서 외부 접근을 위한 유저 생성 및 권한 부여에 대한 간단한 사용법 정리

## 계정 및 권한
MySQL을 설치하면 기본적으로 root 계정이 생성된다.  
root 계정은 모든 데이터베이스에 대해 접근 가능하고, 변경 권한이 있다.  
따라서 root 계정을 공유해서 사용하거나, 계정 정보가 노출이 되었을 경우 임의의 사용자가 모든 데이터베이스의 내용을 조회하거나 변경할 수 있는 문제점이 있다.  
이러한 문제점을 해결하기 위해서 MySQL에서는 계정별로 권한을 부여할 수 있는 기능을 제공하고 있다.

## 계정 생성 및 권한 부여 

### 계정 생성
```shell
mysql> create user 계정ID@'호스트네임/IP' identified by '계정비밀번호';
mysql> create user order@'%' identified by 'order123'; # 예시
```
계정ID, 호스트네임/IP, 계정비밀번호를 지정한다.  
호스트네임/IP자리에 아무값도 넣지않으면 기본값인 '%'이 들어간다. '%'은 모든 호스트네임/IP를 의미한다.

### 권한 부여

```shell
mysql> grant 부여할권한 on DB이름.테이블이름 to 계정ID@'호스트네임/IP';
mysql> grant SELECT on order.* to order@'%'; # 예시 order@'%'에 order DB의 모든 테이블에 대해 SELECT 권한만 부여
```
권한을 부여할 계정 정보, 권한 부여가 필요한 DB와 그 DB의 테이블, 부여할 권한을 지정한다.  
권한을 부여할 계정 정보는 위의 계정 생성 과정에서 생성된 정보를 넣어준다.  
테이블이름 자리에 *을 넣으면 해당 DB의 모든 테이블에 대한 권한이 부여 된다.  
부여할 권한을 지정하는 방법은 공식도큐먼트를 참고하자.


> MySQL 5.7 이하에서는 `grant 부여할권한 on DB이름.DB이름.테이블이름 to 계정ID@'호스트네임/IP' identified by '계정비밀번호';` 명령으로만으로도 계정 생성과 권한 부여가 한번에 된다.
{: .prompt-tip }


## 권한 조회
```shell
mysql> show grants for 계정ID@'localhost' ;
mysql> show grants for order@'%'; # 예시 order@'%'에 부여된 권한 조회
```

## Refercence
* <https://dev.mysql.com/doc/refman/5.7/en/create-user.html/>
* <https://dev.mysql.com/doc/refman/5.67en/grant.html/>
* <https://dev.mysql.com/doc/refman/8.0/en/create-user.html/>
* <https://dev.mysql.com/doc/refman/8.0/en/grant.html/>