---
title: Serivce 테스트
date: 2023-04-23 15:20:42 +0900
categories: [ Spring ]
tags: [ Spring, Test, Service ]
comments: true
---

스프링 웹 애플리케이션에서 Service 계층 테스트

## Service 계층의 역할
layered architecture에서 service 계층은 business 계층으로도 불린다.    
상위 계층인 controller(presentation) 계층으로 부터 요청 받은 동작을 수행하는 역할을 한다.  
service 계층은 하위 계층인 repository 계층의 메소드를 조합해서 business logic을 수행한다.  
service 계층 테스트는 business logic이 잘 동작하는지 테스트한다.

## classicist vs mockist
service 계층 테스트에서는 하위 계층인 repository 계층에 대한 mocking 여부(Test Double 사용여부)를 결정해야한다.  
mocking을 하지 않는다면 service, repository 두 계층에 대한 실제 bean을 로드해서 테스트를 진행한다. 이러한 테스트를 Sociable Unit Test라고 한다.  
반대로 mocking을 하면 service는 실제 bean, repository는 mock bean(Test Double)을 가지고 테스트 하는 것을 Solitary Unit Test라고 한다.  

테스트 신뢰도 측면에서는 Sociable Unit Test가 더 좋다.  
실제 repository를 가지고 테스트하는 Sociable Unit Test는 service에 의해 수행된 repository의 결과를 직접 확인할 수 있고 테스트 데이터에 대해서 실제 애플리케이션과 동일한 동작을 하기때문에 신뢰도가 상대적으로 높아질 수 있다. 하지만 Solitary Unit Test에서는 개발자가 직접 mocking을 하기 때문에 mocking에 대한 오류가 있거나 하는 경우에 대해서 테스트에 대한 신뢰도가 상대적으로 낮을 수 있다는 점이 있다.

테스트 코드 작성면에서는 Solitary Unit Test가 더 간결하다.  
Sociable Unit Test에서는 필요한 객체를 직접 다 생생해주는 과정(Fixture)이 필요하다. 하지만 mocking 과정에서는 given(), any()와 같은 메소드로 아주 간단하게 작성할 수 있다.

상태 검증 위주의 Sociable로 좀 더 신뢰감 있게 테스트를 해야한다고 주장하는 사람들을 classicist,  
행위 검증 위주의 Solitary한 테스트로도 충분하다는 사람들을 mockist라고 한다.

## service 계층 테스팅에서는 classicist가 좀 더 낫지 않을까?
위에서 classicist와 mockist에 대한 설명을 했다.  
둘 중 어느 방식이 더 옳고 나쁘다고 할 수 없다.  
두 방식 모두 장단점이 있고 잘 맞는 환경, 상황등이 있기에 대립되는 방법론이라고 생각한다.  

나는 개인적으로 service 계층 테스트에서는 실제 bean을 주입해서 테스트하는 classicist 방식을 채택하는 것이 좋다고 생각한다.  
service 계층의 로직이 가벼우면 mocking을 처리해서 테스트를 해도 충분하겠지만, 로직이 무거워지면 테스트 코드를 작성하는 사람이 mocking을 완벽하게 하는 것이 쉽지 않기 때문이다.  

또 위에서 언급했듯, 실제 repository bean을 가지고 수행된 결과에 대한 값들을 직접 가지고 상태 검증을 하는 과정이 더 안정감을 줄수있다고 생각한다.  
 
독자들이 간단하게 비교가 가능하도록 본 글에서는 간단한 메소드 하나를 classicist와 mockist 두가지 방식으로 테스트 하는 방법을 소개한다.  

## Base Class (classicist)

```java
@ActiveProfiles("test")
@SpringBootTest
public class ServiceTest {

}
```
- @SpringBootTest  
    - Spring Application Context가 실행된다. Spring 관련 설정(Bean)을 전부 로드한다.
    - 통합테스트시 사용된다.
- @ActiveProfiles  
    - 테스트용 프로파일 지정 (test/resources/applicatoin-test.yml)

## Test Class Code (classicist)
```java
class PostServiceTest extends ServiceTest {
    @Autowired
    private PostService postService;

    @Autowired
    private PostRepository postRepository;

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
        PostResponse postResponse = PostService.createPost(request);

        // then
        assertThat(productResponse)                   // 수행된 값 확인 (상태 검증)
            .extracting("title", "author")
            .contains("테스크 코드 작성법", "sehyeong")

        List<Post> posts = postRepository.findAll();  // 수행된 값 확인 (상태 검증)
        assertThat(posts).hasSize(1);
        assertThat(posts.get(0))
            .extracting("title", "content", "author")
            .contains("테스크 코드 작성법", "본문 내용", "sehyeong");
        
    }
}
```
classicist 방식의 테스트 코드는 [이전글](/posts/repository-test/)에서 다룬 repository의 테스트 코드와 마찬가지로 직접 수행된 결과를 값을 기댓값과 일치하는지 확인해보는 방식으로 작성 할 수 있다. (이러한 방식을 상태 검증이라고 한다.)

## Base Class (mockist)

```java
@ActiveProfiles("test")
@ExtendWith(MockitoExtension.class)
// @SpringBootTest
public class MockTest {

}
```
- @ExtendWith(MockitoExtension.class)  
    - 테스트 클래스에서 mockito framework 사용 명시 및 활성화
- @ActiveProfiles  
    - 테스트용 프로파일 지정 (test/resources/applicatoin-test.yml)



## Test Class Code (mockist)
```java
class PostServiceTest extends MockTest {
    @InjectMocks
    // @Autowired
    private PostService postService;

    @Mock
    //@MockBean
    private PostRepository postRepository;

    @DisplayName("게시글 생성 요청시, 게시글을 저장한다.")
    @Test
    void createPost(){

        // given
        Post expected = Post.builder()
                .title("테스크 코드 작성법")
                .content("본문 내용")
                .author("sehyeong")
                .build();

        PostCreateRequest request = PostCreateRequest.builder()
                .title(expected.getTitle())
                .content(expected.getContent())
                .author(expected.getAuthor())
                .build();

        given(postRepository.save(any(Post.class))).willReturn(expected)

        // when
        PostResponse postResponse = PostService.createPost(request);

        // then
        assertThat(productResponse)                   // 수행된 값 확인 (상태 검증)
            .extracting("title", "author")
            .contains("테스크 코드 작성법", "sehyeong")

        verify(postRepository).save(any(Post.class)); // save 메소드 호출 확인 (행위 검증) 
    }
}
```
- @Mock
    - Mock 객체를 생성한다.
    - 예시 코드에서는 `postRepository = Mockito.mock(PostRepository.class);`와 같은 의미이다.
- @InjectMocks
    - @Mock이 붙은 Mock 객체를 @InjectMocks 객체에 주입해준다.
    - 예시 코드에서는 PostService가 PostRepository Mock 객체를 주입 받도록 함

mock 테스트에서는 repository의 save 메소드가 한번 호출 됐는지에 대한 검증으로 테스트를 마무리한다.  
이렇게 특정한 메소드가 수행되는지에 대한 검증을 하는 것을 행위 검증이라고 한다. 상태 검증과는 달리 단순히 원하는 메소드가 잘 호출 됐는지 정도만 확인하는 것이다.

> `@ExtendWith(MockitoExtension.class)`를 사용하면 Spring의 모든 Bean이 로드 되는 것이 아니여서 Service에서 필요로 하는 객체들을 따로 생성해주거나 불필요한 mocking 작업이 필요할 수 있다. `@SpringBootTest`(모든 Bean 로드) + `@Autowired`(Bean 자동 주입) + `@MockBean`(실제 Bean 대신 MockBean 주입) 조합을 사용할 수 도 있고 이렇게 많이 사용한다.
{: .prompt-tip }


## 정리 

Serivce 계층을 두가지 방식으로 테스트하는 것에 대해 알아보았다.      
해당 글 예시에서는 로직이 비교적 간단한 테스트여서 두가지 방식의 차이가 크게 나지 않아 보일 수 있다.    

혹자는 classicist 방식 처럼 통합테스트 형식으로 실제 환경과 동일한 객체로 테스트하는게 당연하다고 생각이 들 수 도 있고,
또 다른 혹자는 가벼운 로직의 service라면 mock 테스트로도 충분하다고 생각이 들 수 도 있다.  

해당 글을 통해 두가지 테스트 방식에 대한 개념을 이해하는데 도움이 되었길 바라며,  
직접 자신의 프로젝트에 두가지 방식으로 테스트를 해보면서 어떤 방식이 좋을지 고민하고 적용해보자.  


## Refercence
* <https://martinfowler.com/bliki/UnitTest.html/>
* <https://algopoolja.tistory.com/119/>
* <https://github.com/cheese10yun/spring-guide/blob/master/docs/test-guide.md/>
* <https://www.inflearn.com/course/practical-testing-%EC%8B%A4%EC%9A%A9%EC%A0%81%EC%9D%B8-%ED%85%8C%EC%8A%A4%ED%8A%B8-%EA%B0%80%EC%9D%B4%EB%93%9C/>