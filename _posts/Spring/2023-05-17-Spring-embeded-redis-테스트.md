---
title: 스프링에서 embedded redis 테스트 환경 구축하기
date: 2023-05-17 21:11:22 +0900
categories: [ Spring ]
tags: [ Spring, Redis, Test ]
comments: true
---
스프링에서 embedded redis를 사용한 테스트 환경 구성에 대한 글 

## 개요
[이전글](/posts/Spring-redis/)에서 redis를 스프링에서 사용하는 방법에 대해서 간단하게 소개했다.  
RDBMS를 사용하는 JPARepository 테스트 코드에서는 H2 database를 사용하거나, 실제 테스트용 DB를 띄워놓고 테스트 코드를 수행할 수 있다.  
이와 마찬가지로 redis를 저장소로 사용하는 로직의 테스트코드의 동작을 위해서는 테스트용 redis를 띄워놓는다던가, H2 database처럼 임베디드 레디스가 필요하다.  

## h2 db와 같은 내장형 임베디드 데몬 환경이 테스트에 좋은 이유
설치가 간단하고 로컬 환경 구성으로 테스트를 할 수 있다라는 점이 가장 큰 장점이라고 생각한다. 따로 mysql이나 redis를 직접 설치해서 실행해야하고, 도커를 사용한다고 하더라도 도커 명령어를 수행하고 실행하는 등 여러 절차가 필요하다. 하지만 h2 db의 경우 spring boot 프로젝트에서 의존성만 추가하고 설정 정도만 해주면 구성이 끝이난다.  

Embedded redis도 마찬가지로, 의존성을 추가해주고, Embedded redis server를 소스코드로 제어할 수 있다.

## 테스트에 Embeded Redis Config 추가하기
아래 코드를 test 폴더에 넣어주면 된다.  
```java
@Profile("test")
@Configuration
public class EmbeddedRedisConfig {

    private RedisServer redisServer;
    
    @Value("${spring.data.redis.port}")
    private int redisPort;

    @PostConstruct
    public void redisServer() throws IOException {
        int port = isRedisRunning()? findAvailablePort() : redisPort;
        redisServer = new RedisServer(port);
        redisServer.start();
    }

    @PreDestroy
    public void stopRedis() {
        if (redisServer != null) {
            redisServer.stop();
        }
    }

    /**
     * Embedded Redis가 현재 실행중인지 확인
     */
    private boolean isRedisRunning() throws IOException {
        return isRunning(executeGrepProcessCommand(redisPort));
    }

    /**
     * 현재 PC/서버에서 사용가능한 포트 조회
     */
    public int findAvailablePort() throws IOException {

        for (int port = 10000; port <= 65535; port++) {
            Process process = executeGrepProcessCommand(port);
            if (!isRunning(process)) {
                return port;
            }
        }

        throw new IllegalArgumentException("Not Found Available port: 10000 ~ 65535");
    }

    /**
     * 해당 port를 사용중인 프로세스 확인하는 sh 실행
     */
    private Process executeGrepProcessCommand(int port) throws IOException {
        String command = String.format("netstat -nat | grep LISTEN|grep %d", port);
        String[] shell = {"/bin/sh", "-c", command};
        return Runtime.getRuntime().exec(shell);
    }

    /**
     * 해당 Process가 현재 실행중인지 확인
     */
    private boolean isRunning(Process process) {
        String line;
        StringBuilder pidInfo = new StringBuilder();

        try (BufferedReader input = new BufferedReader(new InputStreamReader(process.getInputStream()))) {

            while ((line = input.readLine()) != null) {
                pidInfo.append(line);
            }

        } catch (Exception e) {
        }

        return !StringUtils.isEmpty(pidInfo.toString());
    }

}
```
해당 코드는 [개발자 향로님의 글](https://jojoldu.tistory.com/297)를 사용했다.
[이전글](/posts/Spring-redis/)에서 RedisConfig와 다른 부분은 embedded redis 서버가 실행 중인지 확인하고, 실행 가능한 포트를 확인해서 서버를 실행한다. 이러한 로직이 추가된 이유는 통합 테스트시에 여러개의 스프링 Context가 실행될수있고, 그에 따라서 embedded redis 서버 역시 여러개가 실행되면서 충돌 할 수 있기 떄문이다. 이를 위해서 위와 같은 로직을 추가해줘야 한다.

## 단위 테스트를 위한 RedisRepositoryTest 베이스 클래스
```java
@DataRedisTest
@Import(EmbeddedRedisConfig.class)
@ActiveProfiles("test")
public class RedisRepositoryTest {

}
```
- @DataRedisTest
    - Redis 연결 및 구성을 위한 필요한 Bean들을 자동으로 구성한다. RedisTemplate, StringRedisTemplate, RedisConnectionFactory 등과 같은 Redis 관련 Bean들이 자동으로 설정된다.
- @Import(EmbeddedRedisConfig.class)
    - @DataRedisTest 어노테이션에의해 Spring 전체 context가 로드가 되는 것이 아니기 때문에 EmbeddedRedisConfig가 자동으로 로드가 되지 않기때문에 로드되도록 추가해준다.
- @ActiveProfiles("test")
    - 테스트용 프로파일 지정 (test/resources/applicatoin-test.yml)

## 단위 테스트 예시

```java
public interface RefreshTokenRepository extends CrudRepository<RefreshToken, Long>{
    Optional<RefreshToken> findRefreshTokenByEmail(String email);
}
```

```java
class RefreshTokenRepositoryTest extends RedisRepositoryTest {

    @Autowired
    private RefreshTokenRepository refreshTokenRepository;

    @Test
    @DisplayName("이메일로 refresh token을 조회한다.")
    void findRefreshTokenByEmail(){
        // given
        String email = "test@test.com";
        String token = "token";
        RefreshToken refreshToken = RefreshToken.builder()
                .email(email)
                .token(token)
                .build();
        refreshTokenRepository.save(refreshToken);

        // when
        RefreshToken expected = refreshTokenRepository.findRefreshTokenByEmail(email).orElseThrow();

        // then
        assertThat(expected.getEmail()).isEqualTo("test@test.com");
    } 
```
RefreshTokenRepository의 findRefreshTokenByEmail 메소드에 대한 단위테스트 예시 코드이다.
@DataRedisTest 어노테이션으로 테스트에 필요한 최소한의 context를 로드해서 repository의 메소드에 대한 상태검증 테스트를 진행할 수 있다.

## Refercence
* <https://jojoldu.tistory.com/297/>
* <https://backtony.github.io/spring/redis/2021-08-29-spring-redis-1/#5-dataredistest/>
* <https://devoong2.tistory.com/entry/Springboot-Redis-%ED%85%8C%EC%8A%A4%ED%8A%B8-%ED%99%98%EA%B2%BD-%EA%B5%AC%EC%B6%95%ED%95%98%EA%B8%B0-Embedded-Redis-TestContainer/>
