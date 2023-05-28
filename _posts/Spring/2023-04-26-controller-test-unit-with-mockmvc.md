---
title: Controller 단위 테스트
date: 2023-04-26 19:25:32 +0900
categories: [ Spring ]
tags: [ Spring, Test, Controller, MockMVC ]
comments: true
---

스프링 웹 애플리케이션에서 Controller 계층 단위 테스트

## Controller 계층의 역할
layered architecture에서 controller 계층은 presentation 계층으로도 불린다.  
수신한 client의 요청을 하위 계층인 service 계층에게 위임하는 역할을 한다.

controller 계층에서의 단위 테스트는 수신한 client의 요청이 올바른지 검증하고 API 응답에 대한 검증을 한다.


## Base Class

```java
@WebMvcTest(controllers = {
    PostController.class,
    ...
})
public class ControllerTest {
    @Autowired
    protected MockMvc mockMvc;

    @Autowired
    protected ObjectMapper objectMapper;

    @MockBean
    protected PostService PostService;
    ...
}
```
- @WebMvcTest  
    - Spring Application Context 전체가 아닌 Presentation 계층에 대한 컴포넌트(Bean)만 로드한다.
    - presentation 계층이 아닌 하위 계층의 객체는 @MockBean으로 등록해서 주입받는다.
    - controllers에 테스트 대상 controller 클래스를 넣어준다.
- @MockBean
    - @WebMvcTest를 위한 Spring container에 Mockito의 Mock 객체를 추가해준다.
    - 위 코드에서 @WebMvcTest로 PostService가 로드가 되지 않은 상황이고 이를 Mock 객체화해서 Spring Context에 추가하기 위함
- MockMvc
    - 웹 어플리케이션을 서버에 배포하지 않고 테스트용 MVC환경을 만들어 요청 및 전송, 응답기능을 제공해주는 유틸리티
- ObjectMapper
    - Java Object <-> Json Data 간 변환을 위한 유틸리티

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
endpoint가 `/api/v1/posts`이고, method가 Post인 createPost에 대한 테스트 코드를 작성해보자.

## Test Class Code
```java
@WebMvcTest(controllers = PostController.class)
class PostControllerTest extends ControllerTest {

    @DisplayName("게시글 생성 요청시, 게시글을 저장한다.")
    @Test
    void createPost(){
        // given
        PostCreateRequest request = PostCreateRequest.builder()
                .title("테스크 코드 작성법")
                .content("본문 내용")
                .author("sehyeong")
                .build();

        PostCreateResponse expected = PostCreateResponse.builder()
                .id(1L).build();

        given(postService.createPost(request)).willReturn(expected);

        // when
        ResultActions resultActions = mockMvc.perform(
                        post("/api/v1/posts")
                                .contentType(MediaType.APPLICATION_JSON)
                                .content(objectMapper.writeValueAsString(request)))
                .andDo(print());

        // then
        resultActions
                .andExpect(status().isCreated())
                .andExpect(header().string("Location", "/api/v1/posts/1"));

}
```
mockMvc를 통해서 원하는 endpoint에 요청을 보내고 응답에 대한 상태코드 확인 정도로 작성한다. 앞서 다룬 repository와 service 계층 테스트와 달리 이번 테스트는 하위 계층을 mocking 하고 controller layer만 테스트하는 유닛 테스트의 성격에 가깝다. 글의 서두에서도 언급했듯이 Client의 요청과 API 스펙에 맞는 응답코드 정도를 확인한다. 


## Refercence
* <https://github.com/cheese10yun/spring-guide/blob/master/docs/test-guide.md/>
* <https://www.inflearn.com/course/practical-testing-%EC%8B%A4%EC%9A%A9%EC%A0%81%EC%9D%B8-%ED%85%8C%EC%8A%A4%ED%8A%B8-%EA%B0%80%EC%9D%B4%EB%93%9C/>
* <https://www.baeldung.com/jackson-object-mapper-tutorial/>