---
title: webclient mocking 하기
date: 2023-06-16 15:24:37 +0900
categories: [ Spring ]
tags: [ Spring, WebClient, Test ]
comments: true
---
테스트 코드에서 WebClient의 mocking에 대한 글 (MockWebServer)
## 개요
백엔드 API를 개발하다보면, 외부 HTTP API를 호출하고 그에 대한 결과들을 조합해 응답하는 API를 개발하게 된다.    
OAUTH 로그인 기능, MSA 환경에서 다른 MSA의 API를 호출한다던가 하는 등 꽤나 자주 이러한 상황에 놓이게 된다.    
Spring에서 외부 API 호출을 위해 사용되는 Http Client 라이브러리 중 WebClient를 사용하는 경우 mocking을 어떻게 하는지에 대해 알아보자.  

## Mockito 사용하기
```java
    public String getAccessToken(String code){
        Map<String, Object> responseBody = oauthLoginClient.post()
                .uri(uriBuilder -> uriBuilder
                        .queryParam("code", code)
                        .queryParam("client_id", clientId)
                        .queryParam("client_secret", secretId)
                        .build())
                .retrieve()
                .bodyToMono(new ParameterizedTypeReference<Map<String, Object>>() {})
                .blockOptional()
                .orElseThrow();
        validateResponseBody(responseBody);
        return responseBody.get("access_token").toString();
    }
```

{% raw %}
```java
    @Test
    @DisplayName("authorization code로 access token을 가져온다.")
    public void getAccessToken(){
        // mockWebClient, mockRequestBodyUriSpec, mockRequestBodySpec mockResponseSpec 준비 
        ...

        Map<String, Object> expected = new HashMap<>(){{
            put("access_token","ACCESS_TOKEN");
        }};

        given(mockWebClient.post())
            .willReturn(mockRequestBodyUriSpec);
        given(mockRequestBodyUriSpec.uri(uriBuilder -> uriBuilder
                        .queryParam("code", code)
                        .queryParam("client_id", clientId)
                        .queryParam("client_secret", secretId)
                        .build())
            .willReturn(mockRequestBodySpec)
        given(mockRequestBodySpec.retrieve())
            .willReturn(mockResponseSpec);
        given(mockResponseSpec.bodyToMono(new ParameterizedTypeReference<Map<String, Object>>() {}))
            .willReturn(Mono.just(expected));

        ...
    }
```
{% endraw %}

mockito를 사용해서 WebClient를 mocking하려면 위처럼 모든 chain에 대해서 mocking을 해줘야한다. 매우 불편하고 코드 역시 지저분 해진다.
이러한 단점을 해결하기위해서 WebClient를 mocking하는 것이 아닌 WebClient가 요청하는 서버의 응답을 mocking하는 방법이 있다. 그 방법이 아래서 소개하는 MockWebServer를 사용하는 방법이다.

## MockWebServer 사용하기 
### 1. 의존성 추가하기
```gradle
dependencies {
    testImplementation("com.squareup.okhttp3:mockwebserver:4.11.0")
}
```
{: file='build.grable'}
### 2. Test Base Class
```java
public class MockWebServerTest {
    public static MockWebServer mockBackEnd;

    @BeforeAll
    static void setUp() throws IOException {
        mockBackEnd = new MockWebServer();
        mockBackEnd.start();
    }

    @AfterAll
    static void tearDown() throws IOException {
        mockBackEnd.shutdown();
    }
}
```
### 3. Test Code 예시
```java
class OauthRequesterTest extends MockWebServerTest {

    private ObjectMapper objectMapper = new ObjectMapper();
    private WebClient webClient = WebClient.create();
    private OauthRequester oauthRequester;

    @BeforeEach
    void initialize() {
        String baseUrl = String.format("http://%s:%s", mockBackEnd.getHostName(),
                mockBackEnd.getPort());
        
        oauthRequester = oauthRequester.builder() // Client가 MockWebServer로 요청을 보내도록 준비
        .clientId("clientId")
        .secretId("secretId")
        .oauthUrl(baseUrl)
        .openApiUrl(baseUrl)
        .webclient(webClient)
        .build();
    }

    @Test
    @DisplayName("authorization code로 access token을 가져온다.")
    public void getAccessToken() throws RuntimeException, JsonProcessingException {
        //given
        String authorizationCode = "code";

        OauthAccessTokenResponse expected = OauthAccessTokenResponse.builder()
                .tokenType("bearer")
                .scope("")
                .accessToken("ACCESS_TOKEN")
                .build();

        mockBackEnd.enqueue(new MockResponse()  // MockWebServer의 응답을 Stubbing하는 코드
                .setBody(objectMapper.writeValueAsString(expected))
                .addHeader("Content-Type", "application/json"));

        // when
        String accessToken = oauthRequester.getAccessToken(authorizationCode);

        // then
        assertThat(accessToken).isEqualTo("ACCESS_TOKEN");  // 응답에 대한 검증
    }
}
```
테스트 코드의 큰 흐름을 간단히 설명하면 외부 API를 호출하는 WebClient를 MockWebServer로 요청하도록 준비하고, enqueue 함수를 통해서 응답을 stubbing하는 작업 그리고 MockWebServer의 응답에 대한 검증으로 이루어진다.

> RestTemplate을 사용하고 있다면 MockRestServiceServer을 사용할 수 있다.
{: .prompt-tip }

## Refercence
* <https://www.baeldung.com/spring-mocking-webclient/>
* <https://www.baeldung.com/spring-mock-rest-template/>