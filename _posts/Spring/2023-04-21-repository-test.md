---
title: Repository 테스트
date: 2023-04-21 15:20:42 +0900
categories: [ Spring ]
tags: [ Spring, Test, Repository ]
comments: true
---

스프링 웹 애플리케이션에서 Repository 계층 테스트

## Repository 계층의 역할
layered architecture에서 repository 계층은 persistent 계층으로도 불린다.  
데이터베이스로부터 원하는 데이터를 읽어서 상위 계층인 service(business) 계층으로 올리거나, 상위 계층으로 부터 요청 받은 데이터를 쓰는 역할을 한다.

따라서 repository 계층 테스트에서는 원하는 데이터를 정확하게 데이터베이스로 부터 읽어오는지, 저장하는지를 검증하면 된다. 

## Base Class

```java
@DataJpaTest
@ActiveProfiles("test")
@AutoConfigureTestDatabase(replace=Replace.NONE) // InMemory DB가 아닌 지정한 DB에서 테스트 하고 싶은 경우
public class RepositoryTest {

}
```
- @DataJpaTest  
    - JPA 관련 설정(Bean)만 로드한다.
    - @Transactional을 포함한다.
    - @AutoConfigureTestDatabase을 포함하며 기본적으로 InMemory DB에 테스트를 할 수 있도록 설정된다.
- @ActiveProfiles  
    - 테스트용 프로파일 지정 (test/resources/applicatoin-test.yml)
- @AutoConfigureTestDatabase  
    - 테스트용 프로파일에 DB관련 설정이 있는 경우, Replace.NONE 옵션을 넣어서 해당 어노테이션을 달아준다.    
    - @AutoConfigureTestDatabase의 경우 Default 값이 Replace.ANY이며 테스트에 사용되는 DB가 InMemory DB로 설정된다.   None으로 ImMemory DB로 대체되는 것을 막는 것이다.

## Repository 
```java
@Repository
public interface PostRepository extends JpaRepository<Post, Long> {

    // 여기에 이렇게 추가되는 메소드들에 대해서는 테스트 코드를 작성해주자.
    @Query(value = "select title from post p order by id desc limit 1", nativeQuery = true)
    String findLatestPostTitle();

}
```
Spring Data JPA를 사용할 경우, JpaRepository 인터페이스에 있는 기본 CRUD 관련 메서드는 테스트 하지 않고, 직접 추가한 함수에 대해서만 테스트 코드를 작성해도 충분하다. 

## Test Class Code
```java
class PostRepositoryTest extends RepositoryTest {
    @Autowired
    private PostRepository postRepository;

    @DisplayName("가장 마지막으로 등록된 게시글의 제목를 읽어올 때, 게시글이 하나도 없는 경우에는 null을 반환한다.")
    @Test
    void findLatestPostTitleWhenPostIsEmpty(){
        // gien 

        // when
        String latestPostTitle = postRepository.findLatestPostTitle();

        // then
        assertThat(latestPostTitle).isNull();
    }
}
```
repository의 테스트 코드는 직관적으로 작성할 수 있다.  
마치 계산기 테스트 코드 처럼 직접 수행된 기댓값을 비교하는 로직의 테스트 코드를 작성하면 된다.  
위의 테스트 로직처럼 아무 게시글을 저장하지 않은 상황에서 마지막으로 등록된 게시글의 제목을 조회하면 Null이 반환되는것이 기대되는 로직 그대로를 테스팅하면 된다.  
다른 CRUD 테스팅 로직 모두 given절에서 데이터를 준비해놓고, when절에서 테스트할 메소드를 수행하고, then절에서 원하는 기댓값이 맞는지 확인하는 형태로 직관적으로 작성하면 된다.


## Refercence
* <https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/test/autoconfigure/orm/jpa/DataJpaTest.html/>
* <https://github.com/cheese10yun/spring-guide/blob/master/docs/test-guide.md/>
* <https://www.inflearn.com/course/practical-testing-%EC%8B%A4%EC%9A%A9%EC%A0%81%EC%9D%B8-%ED%85%8C%EC%8A%A4%ED%8A%B8-%EA%B0%80%EC%9D%B4%EB%93%9C/>