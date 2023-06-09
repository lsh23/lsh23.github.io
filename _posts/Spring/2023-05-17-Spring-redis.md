---
title: 스프링에서 Redis 사용하기
date: 2023-05-17 17:09:48 +0900
categories: [ Spring ]
tags: [ Spring, Redis ]
comments: true
---
스프링에서 Redis 사용하기

## 개요
스프링에서 Redis를 사용하는 방법에 대한 글

## Redis 관련 의존성 추가 
```
dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-data-redis'
}
```
{: file=build.gradle'}

## Redis 관련 설정 추가
```java
@Configuration
@EnableRedisRepositories
public class RedisConfig {
    @Value("${spring.data.redis.host}")
    private String host;

    @Value("${spring.data.redis.port}")
    private int port;

    @Bean
    public RedisConnectionFactory redisConnectionFactory() {
        return new LettuceConnectionFactory(host, port);
    }

    @Bean
    public RedisTemplate<String, Object> redisTemplate() {
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(redisConnectionFactory());
        return redisTemplate;
    }
}
```
공식 문서에 따르면 redis 저장소와 Spring의 consistent DAO간 통신을 위한 인터페이스의 역할을 RedisConnection 객체가 담당하고 이 객체는 RedisConnectionFactory 통해서 생성된다.   

해당 예시 코드의 설정에서 사용되는 RedisConnection 구현체 라이브러리는 spring-boot-starter-data-redis에 내장되어 있는 Lettuce이다.

RedisConnection은 low level 단에서 redis 저장소와 통신하면서 binary 값을 주고 받는다면, RedisTemplate은 high level에서 동작한다. Java 객체를 직렬화해서 RedisConnection에 넘겨준다던가와 같은 역할을 한다.

## Entity
```java
@RedisHash(value = "refreshToken", timeToLive = 1209600)
@Getter
public class RefreshToken {
    @Id
    private Long id;

    private String token;

    @Indexed
    private String email;

    @Builder
    private RefreshToken(String token, String email) {
        this.token = token;
        this.email = email;
    }

    public static RefreshToken of(Member member, String refreshToken) {
        return RefreshToken.builder()
                .token(refreshToken)
                .email(member.getEmail())
                .build();
    }
}
```
- @RedisHash(value = "refreshToken", timeToLive = 1209600)
    - value : redis에 저장될 때 key 값이 refreshToken으로 시작된다.
    - timeToLive : redis에 데이터 유지 시간
- @Id 
    - jakarta.persistence.Id가 아닌 org.springframework.data.annotation.Id 어노테이션이다.
    - @RedisHash 어노테이션과 함께 redis에 저장될 때 key 값을 만드는데 사용된다. 
      (e.g.`refreshToken:1678059561678466893`)
- @Indexed
    - secondary indexes 지정
    - secondary index로 조회시 primary index인 id가 조회된다.

## Repository

```java
public interface RefreshTokenRepository extends CrudRepository<RefreshToken, Long>{
    Optional<RefreshToken> findRefreshTokenByEmail(String email);
}
```

JPARepository(RDB)가 아닌 CrudRepository를 상속받도록한다. JPARepository와 마찬가지로 CRUD 메소드를 생성해주고 커스텀한 메소드도 추가할 수 있다.

> CrudRepository가 아닌 직접 redis template을 사용해서 redis에 원하는 데이터를 저장하고 조회 할 수 있다. [공식문서](https://docs.spring.io/spring-data/data-redis/docs/current/reference/html/#redis:template)를 참고하자 
{: .prompt-info }


## 저장 결과 확인
```java
...
    refreshTokenRepository.save(RefreshToken.of(member, refreshToken));
...
```
{: file='AuthService.java'}

```console
127.0.0.1:6379> scan 0
1) "0"
2) 1) "refreshToken:1683537605679172161:idx"
   2) "refreshToken:email:test@test.com"
   3) "refreshToken:1683537605679172161"
   4) "refreshToken"
127.0.0.1:6379> hgetall refreshToken:1683537605679172161
1) "email"
2) "test@test.com"
3) "id"
4) "1683537605679172161"
5) "_class"
6) "com.example.blog.domain.auth.domain.RefreshToken"
7) "token"
8) "eyJhbGciOiJIUzI1NiJ9.eyJlbWFpbCI6InRlc3RAdGVzdC5jb20iLCJpYXQiOjE2ODU1MjM4ODQsImV4cCI6MTY4NTUyNTY4NH0.6MVlqlssJPcHkxqdfTLsjPCGVwkyV1m2bm_XXXXXXXX"
```

## Refercence
* <https://docs.spring.io/spring-data/data-redis/docs/current/reference/html/>

