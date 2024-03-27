# Springboot Test코드

Test 코드를 작성하는 법을 알아보기 전 Test 코드의 필요성에 대해 알아보도록 하겠습니다.

**⇒ 왜 테스트 코드를 작성해야하나 ?**

1. Test 코드를 작성하지 않고 결과를 검증하는 과정은 비용이많이듭니다.
    1. 검증 코드작성
    2. 어플리케이션 실행
    3. PostMan 혹은 Request 요청
    4. log 혹은 print 로 결과 검증
    5. 원하지 않은 결과 발생 시 재작성 재실행 …

### Spring 의 계층 구조

- Controller : 클라이언트의 요청을 받고 클라이언트에게 결과를 반환 (Presentation Layer)
- Service : 비즈니스 로직을 실행하고 결과 반환 (Service Layer)
- Repository : database에 쿼리를 이용해서 CRUD를 하는 계층 (Data Access Layer)
- Domain : Entity 클래스

이렇기에 어플리케이션을 실행해서 Test 를 진행한다면 어느 계층에서 문제가 발생했는지 파악하기에 비용이 드나 계층별 테스트코드를 작성하면 문제의 원인을 쉽게 파악할 수 있습니다.

스프링부트는 서블릿 기반의 웹 개발을 위한 spring-boot-starter-web, 유효성 검증을 위한 Spring-boot-starter-validation 등 spring-boot-starter 의존성을 제공하고 있다.

테스트를 위한 Spring-boot-starter-test 역시 존재하는데 아래와 같은 library 들이 포함되어있다

- JUnit5 : Java application의 **단위 테스트**를 위한 사실상의 표준 테스트 도구
    - Java에서 독립된 단위 테스트를 지원해주는 프레임워크
    - Assert(검증)을 이용해서 결과를 기댓값과 실제 값을 비교
    - @Test 어노테이션마다 독립적으로 테스트가 진행
- Spring Test & Spring Boot Test : 스프링 부트 어플리케이션에 대한 유틸리티 및 **통합 테스트**
- AssertJ : **유연한 검증 라이브러리**
    - **Junit jupiter를 사용하지 않는 이유는 assertJ의 가독성이 좋기 때문입니다. 간단한 기능을 알아보겠습니다.**
    - **1. assertThat은 값 검증에 쓰입니다.**
    - **assertThat(실제값). isEqualTo(기댓값)**
    - **assertThat(실제 객체). isInstanceOf(객체 예상 타입)**
    - **assertThat(실제값). isNull() 등등 실제 값, 값의 타입을 비교하는 여러 연산자들과 쓰이는 메서드입니다.**
    - **2. asserThatThrownBy는 예외 발생 검증에 쓰입니다.**
    - **asserThatThrownBy( () -> 예외를 발생시킬 로직). isInstanceOf(예외 클래스)**
    - **예외가 발생한다면 테스트를 통과하고 발생하지 않는다면 실패하는 메서드입니다.**
- Hamcrest : 객체 Matcher 를 위한 라이브러리
- Mockito : **자바 모킹 프레임워크**
- JSONassert : JSON 검증을 위한 도구
- JsonPath : JSON 용 XPath

### **[ 스프링부트 테스트를 위한 어노테이션 ]**

스프링부트는 spring-boot-test-autoconfigure를 통해 특정 어노테이션을 붙여주면 해당 테스트를 위한 설정들을 자동으로 제공해준다. 대표적으로 다음과 같은 어노테이션들을 제공하고 있다.

- @SpringBootTest : 스프링부트 어플리케이션의 **통합 테스트**를 지원합니다. 이 어노테이션을 사용하면 스프링 어플리케이션 컨텍스트를 전체적으로 로드하여, 어플리케이션의 모든 부분을 함께 테스트할 수 있습니다.
- @WebMvcTest : 스프링 MVC 테스트를 위한 어노테이션입니다. **웹 계층에 대한 테스트를 진행할 떄 사용하며, 컨트롤러를 테스트하는데 특화**되어있습니다. @WebMvcTest를 사용하면 웹에 관련된 빈들만 로드되므로, **웹 계층에 집중하여 빠르게 테스트**를 수행할 수 있습니다.
- @DataJpaTest : 스프링 데이터 JPA테스트를 위한 어노테이션입니다. JPA컴포넌트를 테스트하는데 사용됩니다. 이어노테이션을 사용하면 JPA를 위한 설정만 로드되고, 데이터소스를 자동으로 설정해줍니다.
    - 부연설명 : DataJpaRepository 는 Jpa를 사용하는 Repository에 대한 검증을 수행할 때 사용하는 어노테이션입니다
    - **DataJpaTest 는 Transaction을 포함하고 있어서 1개의 테스트가 끝나면 Rollback 해 다른 테스트에게 영향을 미치지 않습니다.**
    - **DataJpaTeset 로 검증할 수 있는 목록은 DataSource 에 대한 설정 , CRUD가 제대로 작동하는지**
    
- @RestClientTest : REST 클라이언트 테스트를 위한 어노테이션 입니다. REST통신을 테스트하는데 사용되며, MockRestServiceServer를 자동으로 설정하여 서버와의 실제 통신 없이 REST 클라이언트를 테스트 할 수 있게 해줍니다.
- @JsonTest : JSON 직렬화와 역직렬화를 테스트하는데 사용되는 어노테이션입니다. Jackson이나 Gson 등의 JSON 라이브러리를 테스트하는데 사용됩니다
- @JdbcTest : JDBC 테스트를 위한 어노테이션입니다. JdbcTemplate과 같은 JDBC 관련 빈들을 테스트하는데 사용되며, 데이터 소스를 자동으로 설정해줍니다.

**@Autowire 와 @MockBean, @SpyBean**

⇒ 스프링 부트가 제공하는 테스트 어노테이션들을 실제 스프링 컨텍스트를 사용해 빈들을 등록한다. 그러므로 @Autowired로 등록된 빈을 주입받을 수 있다. 특정 객체를 모킹하고 싶은 경우에는 @MockBean 과 @SpyBean 을 사용하면 된다.

- MockBean : 해당 객체를 Mock된 빈으로 등록
- SpyBean : 해당 객체를 spy 된 빈으로 등록

만약 MockBean 으로 선언한 빈이 없다면 Mock 객체를 빈으로 등록하지만, 동일한 타입과 이름의 빈이 존재한다면 해당 빈은 Mock 빈으로 대체된다. 

**@SpyBean 은 스프링 프록시가 적용된 빈이라면 사용이 불가능하니 프록시를 제거해야 한다!** MockBean과 SpyBean의 사용은 테스트 성능에 영향을 줄 수 있다.

## 단위 테스트 장단점

장점 : 

- 새로운 기능에 대해서 빠르게 작성가능
- Test코드 자체가 하나의 문서
- 시간과 비용의 절감

단점 : 

- 독립적인 테스트이므로 다른 객체와 상호작용 처리를 위해서 가짜 객체 정의 필요하다.
- 가짜 객체의 답변 작성 필요함
- 실제 운영 환경과 다른 답변을 내놓을 수 있는 가능성이 있음

### 단위테스트, 통합 테스트 모두 장단점이 명확하며 통합테스트 같은 경우 테스트 비용을 절감할 수 있는 방법이 없다 그래서 단위테스트에 대해 좀더 알아보자

좋은 단위 테스트란 ? 

- 1개의 테스트는 1개의 기능에 대해서만 테스트!
- 테스트 주체와 협력자를 구분하기! (여기서 주체는 테스트를 할 객체이며, 협력자는 테스트를 진행하기 위해 정의하는 가짜 객체이다)
- Given When Then 으로 명확히 구분!
    - **Given: 테스트를 진행할 행위를 위한 사전 준비**
    - **when: 테스트를 진행할 행위**
    - **then: 테스트를 진행한 행위에 대한 결과 검증**

## 통합 테스트 장단점

장점 : 

- 실제 객체를 사용하므로 가짜 객체 사용하지 않아 정의하지 않아도 됨
- 실제 운영 환경과 같은 값을 도출 가능함

단점 : 

- 테스트 하나의 많은 비용이 들어감
- 어느 계층에서 발생한 문제인지 파악하기 힘듬

### @SpringBootTest

→ 위 어노테이션을 사용하면 손쉽게 통합 테스트를 위한 환경을 준비해주며 모든 Bean 들을 스캔하고 어플리케이션 context를 생성하여 테스트를 실행한다.@SpringBootTest 의 어노테이션에는 다양한 값을 줄 수 있는데 아래와같다

- Value 와 Properties : 어플리케이션 실행에 필요한 property를 key = value 형태로 추가할 수 있다.
- args : application 의 arguments 로 값을 전달 할 수 있다.
- classes : 어플리케이션을 로딩할 때 사용되는 컴포넌트 클래스들을 정의할 수 있음
- webEnvironment : 웹 테스트 환경을 설정할 수 있다.

**@SpringBootTest 어노테이션의 webEnvironment**

→ webEnviroment 의 enum 은 총 4가지 값을 가지고 있는데 아래와같다

- Mock
    - 웹 기반의 애플리케이션 컨텍스트를 생성하지만 Mock 환경으로 제공하여 내장 서버가 시작되지 않음
    - 웹 환경이 클래스패스에 없다면 웹이 아닌 어플리케이션 컨텍스트를 생성
    - 웹 기반의 Mock 테스트를 위해 @AutoConfigureMockMvc 또는 @AutoConfigureWebTestClient 와 함께 사용할 수 있음
- RANDOM_PORT
    - 웹 기반의 애플리케이션 컨텍스트를 생성하여 실제 웹 환경을 제공함
    - 내장 서버도 실행되며 사용하지 않는 랜덤 포트를 listen함
- DEFINED_PORT
    - 웹 기반의 애플리케이션 컨텍스트를 생성하고 실제 웹 환경을 제공함
    - 내장 서버도 실행되며 지정한 포트(default 8080)를 listen함
- NONE
    - SppringApplication로 애플리케이션 컨텍스트를 생성함
    - 하지만 mock이나 다른 것들을 포함해 어떠한 웹 환경도 제공하지 않음
    

**@SpringBootTest 의 단점과 슬라이스 테스트(Slice Test)**

⇒ @SpringBootTest 는 기본적으로 모든 빈을 탐색하고 등록한다. 그래서 특정 계층만 테스트가 필요한 상황에서 @SpringBootTest를 사용하면 불필요하게 무거워지고 시간이 오래 걸린다. 그래서 스프링은 특정 부분만 테스트 할 수 있는 Slice Test를 위한 어노테이션들을 제공하며 물론 Slice Test도 스프링 Context를 구성하므로 통합 테스트이다

- 컨트롤러 테스트를 위한 @WebMvcTest 어노테이션

→WebMvcTest

: WebMvcTest 를 사용하면 손쉽게 컨트롤러를 테스트할 수 있는 환경을 준비해주는데, 내장된 서블릿 컨테이너가 랜덤 포트로 실행이된다.

WebMvcTest 는 어프리케이션 컨텍스트를 만들 때 **컨트롤러와 연관된 빈들만을 제한적으로 찾아서 등록**하며 그러므로 일반적 @Component 나 @ConfigurationProperies 빈들은 스캔되지 않는다.

- @Controller, @RestController
- @ControllerAdvice, @RestControllerAdvice
- @JsonComponent
- Filter
- WebMvcConfigurer
- HandlerMethodArgumentResolver
- 기타 등등

추가적인 설정이 필요하면 @Import를 사용할 수 있고, @MockBean이나 @SpyBean 역시 사용할 수 있다. 또한 @WebMvcTest는 컨트롤러 테스트이므로 @WebMvcTest 내부에 @AutoConfigureMockMvc가 들어있다. 그러므로 @Autowired로 MockMvc를 주입받을 수 있으며, 만약 웹플럭스를 이용중이라면 @WebFluxTest를 사용하면 된다.

### **Service 계층 Test**

**Service 계층은 Repository객체를 Spring에게 주입받고 있습니다.**

**따라서 Service계층의 Test는 주체가 Service 객체이며, 협력자는 Repository객체입니다.**

**그렇기에 Repository는 가짜 객체로서 응답을 설정해줘야 합니다.**

**Junit5 기능을 사용하고, Test에서 가짜 객체를 사용하기 때문에 @ExtendWith(SpringExtension.class)를 붙여줘야 합니다.**

```java

@ExtendWith(SpringExtension.class)
public class MemberServiceTest {

// Test 주체
MemberService memberService;

// Test 협력자
@MockBean
MemberRepository memberRepository;

// Test를 실행하기 전마다 MemberService에 가짜 객체를 주입시켜준다.
@BeforeEach
void setUp(){
    memberService = new MemberServiceImpl(memberRepository);
}
}

@BeforeEach: Test를 실행하기 전 항상 실행하도록 하는 어노테이션입니다. 여기서는 가짜 객체를 주입하는 데 사용됐습니다.
@MockBean: 가짜 객체를 만드는 역할을 합니다. 물론 가짜 객체이므로 응답을 정의해줘야 합니다. Test의 협력자 역할을 합니다. 
MemberService: Test의 주체로서 가짜 객체를 주입받고, 자신의 로직을 실행하고 결과를 가지고 검증을 합니다.

@Test
@DisplayName("멤버 생성 성공")
void createMemberSuccess(){
    /*
    given
     */
    Member member3 = Member.builder().name("hi3").age(10).build();
    ReflectionTestUtils.setField(member3,"id",3l);
 
    Mockito.when(memberRepository.save(member3)).thenReturn(member3); // 가짜 객체 응답 정의
    /*
    when
     */
    Long hi3 = memberService.createMember("hi3", 10);
    /*
    then
     */
    assertThat(hi3).isEqualTo(3L);
}

Member 생성을 성공하는 Test입니다.
ReflectionTestUtils.setField() : test를 진행하면서 private로 선언된 필드 값을 넣어줄 수 있습니다. 
Mockito.when(가짜 객체의 로직 실행). thenReturn(실행되면 이것을 반환한다.)라고 말할 수 있습니다.

@Test
@DisplayName("멤버 생성시 member1 과 이름이 같아서 예외 발생")
void createMemberFail(){
    /*
    given
     */
    Member member1 = Member.builder().name("hi1").age(10).build();
    Mockito.when(memberRepository.findByName("hi1")).thenReturn(Optional.of(member1));
 
    /*
    when then
     */
    assertThatThrownBy(() -> memberService.createMember("hi1",10)).isInstanceOf(IllegalStateException.class);
}

assertThatThrownBy는 예외 발생을 검증하는 메서드입니다. 
memberRepository.findByName("hi1"))을 했을 경우 Optional.of(member1)이 반환됩니다.
MemberService내에는 name으로 중복 예외를 터트리는 로직이 있으므로 예외가 발생합니다.
따라서 예외가 발생해서 테스트를 통과하는 모습을 볼 수 있습니다.

```

### **[ JPA 레포지토리 테스트를 위한 @DataJpaTest 어노테이션 ]**

### **@DataJpaTest**

JPA 레포지토리 테스트를 위해서는 @DataJpaTest를 이용할 수 있다. @DataJpaTest는 기본적으로 @Entity가 있는 엔티티 클래스들을 스캔하며 테스트를 위한 TestEntityManager를 사용해 JPA 레포지토리들을 설정해준다. 마찬가지로 @Component나 @ConfigurationProperties 빈들은 스캔되지 않는다.

### **@DataJpaTest의 기본적인 동작**

앞서 설명하였듯 스프링은 테스트에 @Transactional이 있으면 테스트가 끝난 후 자동으로 트랜잭션을 롤백한다. @DataJpaTest에는 @Transactional 어노테이션이 들어있어서 기본적으로 모든 테스트가 롤백된다. 만약 롤백을 원하지 않는다면 @Rollback(false)를 추가하면 된다.

```java
@DataJpaTest
@Rollback(false)
class MyRepositoryTests {

// ...

}
```

또한 만약 H2와 같은 내장 데이터베이스가 클래스 패스에 존재한다면 내장 데이터베이스가 자동 구성된다. spring-boot-test 의존성에는 기본적으로 H2가 들어있으므로 별다른 설정을 주지 않는다면 H2로 설정된다. 내장 데이터베이스로 설정되기를 원하지 않는다면 다음과 같이 AutoConfigureTestDatabase의 replace 속성을 NONE으로 주면 된다.

```java
@DataJpaTest
@AutoConfigureTestDatabase(replace = Replace.NONE)
class MyRepositoryTests {

// ...

}
```

# **3. 스프링 부트 테스트의 애플리케이션 컨텍스트 캐싱**

---

### **[ 애플리케이션 컨텍스트 캐싱 ]**

스프링 부트가 제공하는 테스트는 모두 애플리케이션 컨텍스트를 구성해주어야 한다. 하지만 모든 테스트마다 이를 구성하려면 비용이 커지므로 스프링은 테스트 시에 내부적으로 스프링 컨텍스트를 캐싱해두고 동일한 설정이라면 재사용한다. 그러므로 다음과 같이 애플리케이션 컨텍스트 설정에 변경을 주는 기능들은 테스트 시에 새로운 컨텍스트를 생성하도록 요구한다.

- @MockBean, @SpyBean
- @TestPropertySource
- @ConditionalOnX
- @WebMvcTest에 컨트롤러 지정
- @Import
- 기타 등등

@MockBean과 @SpyBean은 특정 빈을 Mock이 적용된 빈으로 등록한다. 그러므로 애플리케이션 컨텍스트가 갖는 빈이 달라져 새로운 컨텍스트를 생성하게 된다. 그러다보니 @MockBean과 @SpyBean을 많이 사용하면 테스트가 느려질 수 있으며 캐싱된 애플리케이션 컨텍스트의 수를 증가시킨다. 물론 테스트 클래스에서 @MockBean과 @SpyBean이 적용된 객체가 완전히 동일하다면 재사용한다.

그러므로 만약 현재 테스트 속도가 느리다면 테스트들마다 애플리케이션 컨텍스트가 생성되는지 확인해보도록 하자.

![https://blog.kakaocdn.net/dn/Ck2c7/btrz3lbadwF/F4j0eN24a4Cfqz8yA7pJ3k/img.png](https://blog.kakaocdn.net/dn/Ck2c7/btrz3lbadwF/F4j0eN24a4Cfqz8yA7pJ3k/img.png)

### **[ 스프링 부트 테스트 어노테이션 정리 및 요약 ]**

![https://blog.kakaocdn.net/dn/EOb7k/btrzzX2UWF6/vVwN4qwSMUUWBQuGfbFQ20/img.png](https://blog.kakaocdn.net/dn/EOb7k/btrzzX2UWF6/vVwN4qwSMUUWBQuGfbFQ20/img.png)

여기서 주의해야 할 점은 슬라이스 테스트가 단위 테스트는 아니라는 점이다. 해당 어노테이션으로 테스트를 진행하면 테스트를 위한 애플리케이션 컨텍스트가 준비된다. 즉, 스프링이 준비되므로 해당 테스트들은 통합 테스트에 해당한다.

추가로 위에서 언급한 슬라이스 테스트 어노테이션 외에도 @JsonTest, @RestClientTest, @DataJdbcTest 등도 있다. 만약 json 관련 테스트를 위해 gson이나 objectMapper 등의 의존성이 필요하다면 @JsonTest를, RestTemplate이 필요하다면 @RestClientTest를 사용할 수 있다. 또한 Datasource와 JdbcTemplate만 필요하다면 @JdbcTest를 이용하면 된다.

이번에는 스프링부트에서 테스트를 작성하기 위한 다양한 어노테이션(@SpringBootTest, @WebMvcTest, @DataJpaTest)들을 알아보았습니다. 실제 테스트를 작성하는 방법은 [이 포스팅](https://mangkyu.tistory.com/145)을 참고해주세요!

또한 스프링 부트가 제공하는 테스트는 통합 테스트인만큼 주의사항들이 있습니다. 이어지는 포스팅에서는 테스트가 느려지지 않도록 스프링 부트 설정 주의사항과 테스트 코드 작성 시에 참고할만한 내용을 다뤄보도록 하겠습니다.