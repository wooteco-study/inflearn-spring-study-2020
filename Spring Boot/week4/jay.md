# 스프링부트 개념과 활용

## SpringApplication 1부

- Debug 모드로 실행하면 어떤 빈이 들어오는지 보인다
- 배너 바꾸기  .....

## SpringApplication 2부

애플리케이션 리스너는 애플리케이션 컨텍스트가 있어야 된다.

-D 는 jvm option이다.

— 로 쓰는거는 ApplicationArguments

    ApplicationAriguments

ApplicationRunner 애플리케이션 실행 후 뭔가 실행 할 때

## 외부 설정 1부

사용가능한 외부 설정

- properties, yaml
- 환경변수
- command line argument

흔히 쓰는 [application.properties](http://application.properties)는 jar 안에 있는 application.properties 설정방법으로 우선 순위가 15위다.

1. 유저 홈 디렉토리에 있는 spring-boot-dev-tools.properties
2. 테스트에 있는 @TestPropertySource
3. @SpringBootTest 애노테이션의 properties 애트리뷰트
4. 커맨드 라인 아규먼트
5. SPRING_APPLICATION_JSON (환경 변수 또는 시스템 프로티) 에 들어있는
프로퍼티
6. ServletConfig 파라미터
7. ServletContext 파라미터
8. java:comp/env JNDI 애트리뷰트
9. System.getProperties() 자바 시스템 프로퍼티
10. OS 환경 변수
11. RandomValuePropertySource
12. JAR 밖에 있는 특정 프로파일용 application properties
13. JAR 안에 있는 특정 프로파일용 application properties
14. JAR 밖에 있는 application properties
15. JAR 안에 있는 application properties
16. @PropertySource
17. 기본 프로퍼티 (SpringApplication.setDefaultProperties)

Test 환경에서 proeprties를 쓰면 오버라이드가 되었었는데 그게 '우선순위'가 높아서 그랬던 것.

test/resources 에 application.properties를 쓰면 오버라이드가 된다. 왜냐면 src 컴파일 후 test가 진행되면서 다시 덮어쓰기 때문이다.

SpringApplication 을 Run 하면 src 까지만 빌드하고 classpath에 넣고 띄운다.

    @SpringBootTest(properties = "keesun.name=keesun2")
    public class SpringAppTest {
    }

    @TestPropertySource(location = "classpath:/test.propeties")
    public class SpringAppTest {
    }

TestPropertySource는 properties 파일을 직접명시해준다.

## 외부 설정 2부 (1)

같은 prefix로 시작하는 설정들을 한 번에 몰기

    @ConfigurationProperties("keesun")
    public class KeesunProperties {
    }

@DurationUnit

server timeout 쓸 때 유용하게 쓸 수 있을 것 같다.

@Validated 

로 프로퍼티의 값 검증을 할 수 있다.

## 프로파일

    @Profile("prod")
    @Configuration
    public class BaseConfiguration {
    }

[spring.profiles.active](http://spring.profiles.active) 로 활성화활 프로퍼티를 사용할 수 있다.

## 로깅 1부: 스프링 부트 기본 로거 설정

로깅 퍼사드 vs 로거

로거 api들을 추상화 해놓은 인터페이스들 Commons Logging, SLF4j

주로 프레임워크는 로깅 퍼사드를 이용한다.

Spring Core 가 만들어질 때 스프링개발자들이 이미 Commons Logging을 쓰고 있었다. 

Commons Logging에는 런타임때 로거를 찾는다. 최종적으로는 logback이 찍는다. 로그백이 SLF4j의 구현체이다. 

logging.path 로 로깅 경로를 남길 수 있다. 

## 로깅 2부: 커스터마이징

logback-spring.xml

으로 커스텀한 로그 포맷을 지정할 수 있다.

## 테스트

    @RunWith(SpringRunner.class)
    @SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.MOCK)

MOCK: 내장 톰캣 구동하지 않음. 그리고 mockMVC를 사용한다.

RANDOM_PORT, DEFINED_PORT: 내장 톰캣 사용

NONE: 서블릿 환경 제공 하지 않음

### TestRestTemplate

resttemplate과 같은 방식으로 호출 후 테스트 가능

@MockBean

으로 해당 빈을 mocking 할 수 있다.

### WebTestClient

기존의 restemplate은 sync 이고 webtestclient는 async하게 동작할 수 있다. 이거를 사용하려면 webflux dependency를 추가해야 한다. 

@SpringBootTest는 통합테스트이다. 빈을 전부 찾아서 띄운다. 찾아서 mock이면 mock으로 대체한다. 

슬라이스 테스트를 하려면 

@JsonTest, @DataJpaTest, @WebMvcTest

webmvctest는 컨트롤러 하나만 테스트할 수 있다. 웹과 관련된 것들만 등록시켜준다. 일반적인 component는 등록되지 않는다. 의존성이 싹 끊긴다. 사용하는 의존성이 있으면 mock으로 채워넣어야 한다. mockMvc와 mockBean을 사용.

## 테스트 유틸

OutputCapture

    @Rule
    public OutputCapture 생성

JUnit 꺼다. 콘솔의 로그를 가져온다. 로그검증을한다. 

## Spring-Boot-Devtools

스프링부트가 제공하는 옵셔널한 툴이다.

캐시 설정을 개발환경에 맞게 변경

클래스패스에 있는 파일이 변경될 때 마다 재시작

LiveReload

전역설정은 spring-boot-devtools.properties에서

## 스프링 웹 MVC 1부: 소개

SpringBootAutoConfigure 에 WebMvcAutoConfigurer 가 자동설정되어있다. 

HiddenMethodFilter 는 DELETE, PUT 핸들링 가능.

SpringBoot가 주는 web설정을 확장하려면 WebMvcConfigurer를 구현받아서 수정한다.

@EnableWebMvc를 해버리면 전부다 새로 구현해야 한다. 

## 스프링 웹 MVC 2부: HttpMessageConverters

@ResponseBody로 객체 내보낼 때 (즉 http 요청 본문을 객체로 변경하거나, 객체를 http 응답본문으로 변경할 때)

기본적으로 JsonMessageConverter가 사용된다. 그냥 문자열 String 이라면 StringMessageConverter가 사용된다.

## 스프링 웹 MVC 3부: ViewResolve

accept 헤더에 따라서 응답이 달라진다. 

다른 메세지 컨버터가 필요하면 의존성에 추가한다. (예를들어 xml)
