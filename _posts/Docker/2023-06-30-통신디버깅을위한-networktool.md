---
title: 컨테이너간 통신 디버깅을 위한 networktool
date: 2023-06-30 18:47:06 +0900
categories: [Docker]
tags: [Docker]
comments: true
---

컨테이너간 통신을 확인할 떄 사용하면 좋은 networktool에 대한 간단한 소개글

## 개요

컨테이너를 여러개 띄워 놓고 서로 통신하는 환경을 구성하다보면 통신이 안되서 통신을 확인해야하는 경우가 있다.

하지만 대부분의 이미지가 경량화되어있기때문에 network 관련된 cli(ping, telnet 등) tool을 설치가 되어있지 않다.  
해당 컨테이너 이미지의 베이스 os에 따라서 apk, apt, yum과 같은 패키지 매니저로 설치를 해서 사용할 수 있으나 매번 설치하기도 번거롭고 심지어는 패키지 매니저가 포함되어있지 않은 이미지들도 있다. 이런 경우 network cli를 구성하기가 난감하다.

이떄 사용할 수 있는 것이 이번 글에서 소개하는 network cli만 모아서 설치해둔 컨테이너 이미지이다.

## network-multitool

docker hub : <https://hub.docker.com/r/praqma/network-multitool/>

```shell
docker pull wbitt/network-multitool
docker run -d --name net-tool wbitt/network-multitool
docker exec -it net-tool /bin/bash
bash-5.1# telent 192.168.0.11 3306
```

wbitt/network-multitool를 컨테이너로 띄우고 터미널을 붙여서 network cli를 사용하면 된다.  
curl, wget, dig, nslookup, route, traceroute, ping, telnet, tcpdump등 웬만한 tool은 다 포함되어있다.
더 자세한 내용은 위의 docker hub repo를 확인해보자.

## docker-compose에서 사용하기

```yml
version: "3"
services:
  net-tool:
    image: wbitt/network-multitool

  db:
    image: mysql
    environment:
      MYSQL_DATABASE: test_db
      MYSQL_ROOT_PASSWORD: root
    ports:
      - "13306:3306"

  redis:
    image: redis
    ports:
      - "16379:6379"

  web:
    build: .
    entrypoint: java -Dspring.profiles.active=dev -jar /app.jar
    restart: on-failure
    ports:
      - "18080:8080"
    depends_on:
      - db
      - redis
```

docker-compose처럼 동일한 docker network내에서 통신을 확인할 때 유용하다.  
docker-compose up을 하면 net-tool 컨테이너가 해당 네트워크내에 생성이 되기 때문에 통신여부를 확인하기가 용이하다.

예를 들어서 web 컨테이너 내부의 웹 애플리케이션이 초기 실행시에 db나 redis 연결로 오류가나서 계속 죽는 상황인 경우에 net-tool 컨테이너로 접속해서 net-tool컨테이너와 db, redis 간 통신을 확인해서 통신 오류 인지 endpoint 관련 설정 오류인지등과 같은 디버깅 작업을 할 수 있다.


