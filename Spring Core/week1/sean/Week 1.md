# Week 1

# Section 0 - 소개

### 학습목차

- IoC 컨테이너와 빈
- 리소스
- Validation
- 데이터 바인딩
- SpEL
- 스프링 AOP
- Null-Safety

# Section 1 - IoC 컨테이너와 빈

### QA

- maven의 target Directory ? ← gradle에선 어디로 가는걸까
    - 스프링과 부트의 차이인건가 ?

![Week%201/Untitled.png](Week%201/Untitled.png)

### 더 알아볼 것

- 라이프 사이클 인터페이스
- 옵저버 패턴 (이벤트)

### 필기

- `@Autowired`
    - 빈 등록 시 주입하려는 빈을 못 찾을 경우, 생성자 주입이라면 어쩔 수 없지만, 필드/새터 주입이라면 `required=false`로 줘서 빈 주입은 제대로 못해도 일단 해당 빈을 만들 수 있긴 하다.

- ApplicationRunner 구현받은 클래스
- BeanPostProcessor
    - @PostConstruct 와 같은 애노테이션을 써서 빈 라이프 사이클에 개입할 수 있음

![Week%201/Untitled%201.png](Week%201/Untitled%201.png)

- `@ComponentScan`
    1. `basePackages` = "foo.somepackage"

        요즘은 IDE가 많이 도와주지만, 엄밀히 type safe하진 않다.

    2. `basePackageClasses` = DemoApplication.class

        DemoApplication.class가 위치한 패키지부터 스캔한다.

    - Functional Bean Regis- (Spring 5 ~)

        **기존의 Reflection과 Proxy, CGLib를 활용한 빈 등록 방식**은, 빈 등록할 게 많으면 애플리케이션 '구동시'의 성능에 영향을 준다. 구동 후엔 싱글톤으로 상관 없겠으나, 이에 대한 대안책으로 스프링 5 버전부터 추가된 **Functional한 방법의 빈 등록**이 있다.

- 스프링 부트 애플리케이션을 실행하는 방법 3가지
    1. static 방식 (익숙한, 기본)
    2. 인스턴스 생성 방식
    3. 빌더 방식

![Week%201/Untitled%202.png](Week%201/Untitled%202.png)

![Week%201/Untitled%203.png](Week%201/Untitled%203.png)

- 빈의 스코프
    - 싱글톤 - 기본, 애플리케이션 전반에 걸쳐 모두 같은 하나의 인스턴스
    - 프로토타입 - 매번 새로운 인스턴스

        ![Week%201/Untitled%204.png](Week%201/Untitled%204.png)

- scoped-proxy
    - `@Scope(value = "prototype", proxyMode = ScopedProxyMode.TARGET_CLASS)`

        강의문서 그림에서 알 수 있듯이, 프로토타입을 감싼 빈을 보게해서 해결한다.

        이때 CGLib(서드파티긴 함)를 이용. CGLib는 클래스도 프록시로 만들 수 있다. 반면 자바 jdk의 Dynamic Proxy는 인터페이스만 프록시로 만들 수 있다.

- 싱글톤 스코프 주의점
    - 프로퍼티가 공유되므로, thread-safe하게 코딩하라
    - ApplicationContext 초기 구동시 인스턴스가 생성된다

- Property

    property를 추가하는 방법은 다양하다. 물론 중복된 경우 이들 사이에서도 우선순위가 있음

    - -Dkey="value" 처럼 JVM 시스템 프로퍼티로 추가
    - @PropertySource를 이용해 Environment를 통해 추가
        - 직접 environment.getProperty("value")로 직접 꺼내도 되고
        - @Value("${value}"처럼 필드 애노테이션을 이용해도 되고
    - ~

    → 이에 대한 자세한 내용은 `스프링 부트 강좌`를 보자

- Reload 기능이 있는 MessageSource,  `new ReloadableResourceBundleMessageSource()`  예제
    - 애플리케이션 구동중에도 [messages.properties](http://messages.properties) 파일 수정후 빌드하면, 수정된게 적용된다

- Resource 읽어오기

    `resourceLoader.getResource("classpath:test.txt")`

    그냥 "test.txt" 라고 읽어오는 것 보다, classpath, file 같은 접두사를 붙여주면 더 명시적이라서 좋다.

![Week%201/Untitled%205.png](Week%201/Untitled%205.png)

- Validation 추상화
    - 우테코 레벨2, 한창 spring  boot, web mvc로 개발하며 validation하며 삽질했을 때 봤으면 좋았을 챕터
    - LocalValidatorFactoryBean을 기본으로 등록해주니까, 내가 아는 그 기본적인 컬럼 애노테이션들을 사용해 검증할 수 있다. 하지만 #validate()에서 복잡한 비즈니스 로직으로 검증해야한다면, 직접 Validator 구현체를 만들어 사용할 수 도 있을 것.

![Week%201/Untitled%206.png](Week%201/Untitled%206.png)