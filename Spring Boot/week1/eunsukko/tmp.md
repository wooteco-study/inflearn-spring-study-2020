### 스프링 부트???
- 왜 배워야할까?
- 무엇을 얻어가야할까?

- 학습목표
    - 스프링부트의 원리
- 학습방법
    - 코딩을 직접해가면서 짜보기

### 스프링의 원리
- 운영수준을 빠르게 구축하도록 도와줌
- 가장 널리 쓰인다고 생각되는 설정을 제공해줌
    - 써드파티 라이브러리 또한 제공
    - ex. 톰캣 설정
- 자주쓰이는 것 제공 + 쉽게 바꿀 수 있도록
- xml 설정을 더이상 쓰지 않는다..!
    - 코드 생성도 하지 않는다..!


#### 느낀점?
오홍.. system requirements 를 살펴봐야 겠구먼

### 스프링 부트 시작하기
- gradle 의 경우
    -  java -jar ./build/libs/spring-boot-principle-1.0-SNAPSHOT.jar 

- 어떻게 수많은 의존성이 들어온건지...
- 설정파일은???
    - 어떻게 숨겨져있는건가요??!!!
- SpringApplication.run?
    - 어떻게 내장 톰캣이 뜬건가??!
[문서](https://docs.spring.io/spring-boot/docs/)

- 스프링 프로젝트의 구조??
    - SpringApplication
        - 추천하는 패키지 위치가 있음 (default 패키지 두기)
            - 컴포넌트 스캔이 어떤식으로 적용될지 한 번 생각해보면 이유가 짐작이 갈듯

### 스프링 부트 원리..!

#### 의존성 관리 이해

[문서](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#using-boot-dependency-management)

#### 의존성 관리 응용
- 음... gradle 은 의존성을 어떻게 잘 확인할 수 있을랑가?

- mvn repository 에서 검색하기
- 스프링 부트가 지원해주지 않는 라이브러리 추가할 때
    - 버전 명시..!!
        - 개발, 배포할 때.. 각각이 같은 버전을 사용하도록
        - 부트에서 지원해주는 애들은 부트에서 버전을 명시하는 효과가 있는 것임 (파일에선 보이지 않아도)

#### 자동 설정 이해
> 무엇을 해주는 건지 파악을 해봅시다


- @SpringBootApplication
    - @SpringBootConfiguration
    - @ComponentScan
        - 빈 등록 1단계!
        - 컴포넌트 어노테이션이 달린 애들을 스캔하면서 등록
    - @EnableAutoConfiguration
        - 빈 등록 2단계!
        - 이것도 configuration..! (빈을 등록하기위한)
        - 각 구현 클래스들이 @Configuration 을 구현해놓음
            - spring.factories 에서 등록된 클래스들을 확인할 수 있음
            - ex. SpringDataWebAutoConfiguration
        - 많은 애들이 자동으로 등록됨
            - 대신 각각이... conditional 을 통해서... 특정 경우에, 해당 빈이 등록될 수 있도록 되어있음


- 개요
    - @EnableAutoConfiguration (@SpringBootApplication 안에 숨어 있음)


- 궁금한 것들
    - 음... 그렇다면 autoconfigure 패키지?는 무슨 역할을 하는걸까?
        - 일단 설정 정보를 담고있는 구현 클래스(@Configruation 을 달고있는)를 가짐
        - properties 를 통해서.. 설정시 필요한 것들도 가지고 있음...
        - 그렇다면... 이 패키지를 gradle 같은데서 포함시키면... 어떻게 여기에 있는 설정들이 스프링에 등록이 되는걸까? 시작 포인트를 모르겠다
            - 등록된 jar 에서 메타정보인 spring.factories 에서 포함된 클래스들을 자동으로 등록..!

#### 자동설정 만들기..!
[문서](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-developing-auto-configuration)
##### starter 와 autoConfigure
- Xxx-Spring-Boot-Autoconfigure 모듈: 자동 설정
- Xxx-Spring-Boot-Starter 모듈: 필요한 의존성 정의
- 그냥 하나로 만들고 싶을 때는?
    - Xxx-Spring-Boot-Starter

- XxxRunner
    - 스프링 띄웠을 때.. 빈으로 등록되고.. 자동으로 실행되길 원하는 애들

- 따라해보기~
    - mvn install
        - 다른 프로젝트에서도 사용할 수 있도록 로컬 메이븐 저장소에 등록함
        - [gradle 은 우찌할까..](https://proandroiddev.com/tip-work-with-third-party-projects-locally-with-gradle-961d6c9efb02)
            - compile 로하면 Holoman 이 import 되는데... implementation 으로 하면 안된다..  
                - 둘의 차이가 무엇일까? (gradle 공부가 필요하다...)

- 느낀점
    - 05:24 직전
        - 가족들은 놀러갔는데.. 혼자 강의만들고 계심
            - 아.. 이게 개발자의 삶인가...

##### @ConfigurationProperties
> 문제가 있다...! @ComponentScan -> @AutoConfiguration 으로 빈이 등록됨
> 내가 적용한 빈 (@ComponentScan 으로 넣어준) 이 덮어씌워짐..ㅠㅠ

오홍.. @Conditional 을 사용하는 이유?!

- 따라해보기~

- 자동설정 원리??!
    - 자동설정이 어디에 있으며
    - 어떻게 찾아보는지
    - 어떻게 적용되는지
    - 이제 일하면서... 살펴봅시다..!
        - @EnableConfigurationProperties 는 종종 본 것 같은데??

#### 내장 웹 서버 이해
> 스프링 부트는 웹서버가 아니다..!
- 그냥 스프링 프레임워크를 쉽게 사용할 수 있게 해주는 것

- 디스패쳐 서블릿
    - 서블릿 컨테이너들은 바뀔 수 있기 때문?
        - 아따... 서블릿.. 전체적인 구조... 까먹었다..ㅋㅋㅋㅋㅋㅋㅋㅋㅋㅋㅋㅋㅋㅋㅋㅋㅋㅋ

- 스프링 부트는 내장 웹 서버를 쉽게 사용할 수 있도록 해주는 것
    - 자동 등록

#### 내장 웹서버 응용 

##### 컨테이너와 포트
> 톰캣 말고 다른 서블릿 컨테이너를 사용한다면?

- gradle 에서는 어떻게 하려나???

- 강의에서는...
    - 서블릿 컨테이너 적용
    - 포트 설정해보기
        - 랜덤 포트.. 코드에서 얻어내기

##### Https 와 Http2

http 커넥터는 기본적으로 하나
- 그래서 https 를 적용하면 http 가 안되는 것

http2 를 쓰려면 기본적으로 ssl 이 적용되어 있어야 함
