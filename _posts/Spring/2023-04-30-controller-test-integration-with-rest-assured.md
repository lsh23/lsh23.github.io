---
title: Controller 통합 테스트
date: 2023-04-30 19:43:25 +0900
categories: [ Spring ]
tags: [ Spring, Test, Controller, RestAssured ]
comments: true
---

스프링 웹 애플리케이션에서 Controller 계층 통합 테스트

## Controller 계층의 역할
layered architecture에서 controller 계층은 presentation 계층으로도 불린다.  
수신한 client의 요청을 하위 계층인 service 계층에게 위임하는 역할을 한다.

controller 계층에서의 통합 테스트는 실제 Spring Application과 동일한 환경에서의 요청에 대한 응답 결과를 검증한다.
이를 위해서 rest assured를 활용한다.


## RestAssured 이란

Java에서 REST API 테스를 쉽게 할 수 있는 도구이다.
Java DSL[^DSL] 형태로 매우 직관적인 형태로 테스트 코드를 작성할 수 있다.  
아래 코드의 예시처럼 BDD[^BDD]스타일 로 요청 및 응답에 대한 검증을 할 수 있다.


```java
RestAssured
    .given().log().all()
        .contentType(MediaType.APPLICATION_JSON_VALUE)
        .body(PostCreateRequest)
    .when()
        .post("/api/v1/posts")
    .then().log().all()
        .statusCode(HttpStatus.CREATED.value())
```

## RestAssured 설정 
build.gradle에 dependencies에 사용할 version만 추가해주면 쉽게 사용할 수 있다.
```gradle
dependencies {
    testImplementation 'io.rest-assured:rest-assured:5.3.0'
}
```
{: file='build.gradle'}


## Base Class

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class AcceptanceTest {
    @LocalServerPort
    int port;

   @BeforeEach
   public void setUp() {
       if (RestAssured.port == RestAssured.UNDEFINED_PORT) {
           RestAssured.port = port;
       }
   }
}
```
- @SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
    - 임의의 포트번호로 테스트용 SpringBoot 서버를 띄운다.
- @LocalServerPort
    - RANDOM_PORT로 정해진 포트 번호가 port변수로 주입된다.
- setUp()
    - SpringBootTest를 RANDOM_PORT로 서버를 띄우기 때문에 restAssured Client가 요청을 보낼 Port를 지정한다.

## Controller 

```java
@RestController
@RequiredArgsConstructor
public class PostController {

    private final PostSerivce postService;

    @PostMapping("/api/v1/posts")
    public PostResposne createPost(@Valid @RequestBody PostCreateRequest request){
        return postService.createPost(request);
    }

}
```
enpoint가 `/api/v1/posts`이고, method가 Post인 createPost에 대한 테스트 코드를 작성해보자.

## Test Class Code
```java
class PostAcceptanceTest extends AcceptanceTest {

    @DisplayName("게시글 생성 요청시, 게시글을 저장한다.")
    @Test
    void createPost(){
        // given
        PostCreateRequest request = PostCreateRequest.builder()
                .title("테스크 코드 작성법")
                .content("본문 내용")
                .author("sehyeong")
                .build();

        
        // when
        ExtractableResponse<Response> extract =
                given().log().all()
                    .contentType(MediaType.APPLICATION_JSON_VALUE)
                    .body(request)
                .when()
                    .post("/api/v1/posts")
                .then().log().all()
                .extract();

        // then
        assertThat(extract.statusCode()).isEqualTo(HttpStatus.CREATED.value());
        assertThat(extract.header("Location")).isEqualTo("/api/v1/posts/1");
    }
}
```
mockMvc와 마찬가지로 원하는 endpoint에 요청을 보내고 응답에 대해 검증한다.  
mockMvc를 이용한 테스트에서는 하위 레이어가 테스트 더블(가짜 객체)로 구성이 되어있었지만, 이번 테스트 코드는 통합테스트이면서 실제 Apach Tomcat 환경에 대한 요청에 대한 E2E[^E2E] 성격의 세스트이다. 


## Refercence
* <https://github.com/cheese10yun/spring-guide/blob/master/docs/test-guide.md/>
* <https://www.inflearn.com/course/practical-testing-%EC%8B%A4%EC%9A%A9%EC%A0%81%EC%9D%B8-%ED%85%8C%EC%8A%A4%ED%8A%B8-%EA%B0%80%EC%9D%B4%EB%93%9C/>
* <https://www.baeldung.com/jackson-object-mapper-tutorial/>
* <https://rest-assured.io//>

[^DSL]: Domain Specific Languages 특정 비지니스 도메인을 위한 특수 프로그래밍 언어
[^BDD]: [Behavior-driven development](https://en.wikipedia.org/wiki/Behavior-driven_development)
[^E2E]: [End To End Test](https://circleci.com/blog/what-is-end-to-end-testing/)