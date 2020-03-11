# 의존성 관리 이해

스프링 부트의 그 많은 의존성들은 도대체 어디서 오는가?!

스프링 부트 프로젝트를 생성할 때 우리는 몇 개 안되는 의존성 들을 추가한다.

예를 들어 아래와 같은 것들이 있다.

- spring-book-starter-web
- spring-boot-starter-test

실제로 spring-boot-starter-test 이거 하나만 추가해도 이렇게 굉장히 많은 의존성들이 따라서 들어온다.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8486507a-442b-47e6-b282-2257288b76dc/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8486507a-442b-47e6-b282-2257288b76dc/Untitled.png)

이렇게 하는 게 좋은 점은 

- 우리가 관리해야 될 의존성이 줄어든다.
    - 스프링 버전과 각각의 의존성들의 버전을 맞출 필ㄷ요가 많이 줄어들었다.
    - 특별히 원하는 버전이 있을 때는 버전을 같이 적어주면 해당 버전으로 들어오게 된다.

# 의존성 관리 응용

## 의존성을 추가하는 방법

[mvnRepository.com](http://mvnrepository.com) 에서 원하는 의존성을 검색해서 가져오면 된다.

ModelMapper를 사용하면 dto와 entity간의 변환을 간단하게 해줄 수 있다.

# 자동 설정 이해

@SpringbootApplicaiton은 3가지 Annotation을 포함하고 있다.

- SpringBootConfiguration
- EnableAutoConfiguration
- ComponentScan

### 빈 등록 단계

빈은 사실 두 단계로 나눠서 읽힌다.

- 1단계: @ComponentScan
- 2단계: @EnableAutoConfiguration

### ComponentScan

ComponentScan이란 Component 어노테이션을 가진 클래스들을 Scan해서 Bean으로 등록하는 작업을 말한다.

해당 클래스로가 속한 패키지 부터 하위 모든 패키지 까지를 스캔한다.

### EnableAutoConfiguration

- gradle 프로젝트들 중 ~autoconfigure로 끝나는 파일들이 있다. 이 폴더를 열어서 자세히 보면 META-INF 폴더가 있고, spring.factories라는 파일이 있다. 이 파일에 key, value의 형태로 autoConfiguration들이 정의되어 있다.
    - 여기서 value들 하나 하나가 전부다 Configuration 클래스다.
    - Configuration 클래스는 스프링의 자바 설정 파일이다.
    - 실제로 여기서 value들에 있는 모든 클래스들이 등록되는 것은 아니다.
    - @ConditionalOn~, @AutoConfigure~ 어노테이션들에 따라서 해당 Configuration이 등록될 지 말지 달라진다.

이 때 ComponentScan에서 등록했던 빈이 있다면 덮어쓴다. 따라서 덮어 쓰지 않으려면 ConditionalOn~ 등을 사용하면 된다.

그래서 스프링 부트의 autoConfiguration의 빈들을 보면 ConditionalMissingBean 어노테이션이 많이 붙어있다. 

이 말은 우리가 이 빈을 재정의해서 사용할 때 덮어 쓰지 않게 하기 위함이다.

근데 이렇게 빈을 재정의하면 클래스 생성부터 하나하나 다 해줘야 된다. 

만약! 값만 바꾸고 싶은데 이렇게 클래스를 하나하나 재정의해야 된다면?! → 매우 귀찮다.

그래서 이럴 땐 Properties 파일을 활용하는 방법이 있다.

우리가 매 번 .properties에 적어주는 많은 값들이 이걸 활용한 거라고 볼 수 있다.

- @ConfigurationProperties("alswns")

    @ConfigurationProperties("alswns")//여기에 적어주는 값은 properties 에 적을 때
    //prefix로 적어주는 값이다.
    public class AlswnsProperties {
    	private String name;
    	private int howLong;
    	//Getter, Setter
    }
    

- @EnableConfigurationProperties(AlswnsProperties.class)

위의 두 개의 어노테이션을 활용하면 된다.

    @Configuration
    // AlswnsProperties 파일을 사용하겠다.
    @EnableConfigurationProperties(AlswnsProperties.class)
    public class AlswnsConfiguration {
    		
    	@Bean
    	@ConditionalOnMissingBean -> 값이 다른 클래스가 아니라 완전 다른 클래스가 재정의 됐을때를 
    	//위해서 이렇게 해준다.
    	public Alswns alswns(AlswnsProperties properties) {
    		Alswns alswns = new Alswns();
    		alswns.setHowLong(properties.getHowLong());
    		alswns.setName(properties.getName());
    		return alswns;
    	}
    }

# 자동 설정 만들기

- xxx-Spring-Boot-AutoConfigure 모듈 : 자동 설정
- xxx-Spring-Boot-Stater 모듈: 필요한 의존성을 정의해 놓았다.

그냥 하나로 만들고 싶을 때는?

- xxx-Spring-Boot-Starter

- 삼겹살을 코스트코에서 17불 수고 사오셨지만 너무 많다.
- 3줄을 드셨지만,,,,, 매끼니 먹어도 일주일 동안 먹을 수 있을 만큼 많다.
- 2만원에 일주일치 삼겹살이라니,,,,,, 싸구만

# 내장 웹 서버 이해

- 연보락색 티셔츠를 입으셨다. → 수업의 화사한 분위기를 위해서
    - 는 구라고 아무거나 입으신다. 티셔츠 쌓아놓은 곳에서 그냥 끄내 입으신다.
- 스프링 부트는 서버가 아니다
    - 톰캣, 제티, 네티 같은 것들이 서버이다.
        - 그리고 이런것들은 전부 자바 코드로 서버를 만들 수 있는 기능을 제공한다.
            - 톰캣 객체를 생성
            - 포트를 설정
            - 톰캣에 컨택스트 추가
            - 서블릿 만들기
            - 톰캣에 서블릿 추가
            - 컨텍스트에 서블릿 매핑 → URL 정의해주는 느낌
            - 톰캣 실행 및 대기
    - starter-web을 넣으면 자동으로 tomcat에 대한 의존성이 들어온다
- 이 모든 과정을 보다 상세히 또 유연하게 설정하고 실행해주는게 바로 스프링 부트의 자동 설정
    - ServletWebServerFactoryAutoConfiguration(서블릿 웹 서버 생성)
        - TomcatServletWebServerFactoryCustomizer(서버 커즈터 마이징)
    - DispatcherServletAutoConfiguration
        - 서블릿 만들고 등록

    # 내장 웹 서버 응용

    ## 서블릿 컨테이너 교체하기

    기존에 있던 tomcat이 아니라 다른 서블릿 컨테이너를 사용하고 싶다면?

    - tomcat을 의존성에서 exclusion 해주기
    - jetty, netty 등 원하는 의존성 추가하기

    ### 포트 설정

    - properties 파일에서 `server.port=100`과 같이 정의해주면 된다. 랜덤 포트를 사용하고 싶을 때는 `server.port=0` 이라고 해준다.
    - 톰캣을 만들 때 Connector를 추가할 수 있는데 이 Connector를 추가하면 여러 개의 포트에서 요청을 처리할 수 있다. 따라서 https, http를 모두 처리하려면 http용 커넥터를 추가하면 된다.

    # 독립적인 실행 가능한 jar

    우리가 로컬에서 어플리케이션을 실행할 때는 인텔리제이에서 main 메서드를 실행한다.

    하지만 배포할 때나 도커를 사용할 때는 그렇게 할 수 없다.

    이 때는 jar 파일로 패키징을 하는 방법이 유용하다.

    - /out 폴더는 뭐고 /build 폴더는 뭔가여? 아 그리고 target은 뭔데!

        /build 폴더는 gradle로 빌드하였을 때 생기는 폴더이다. 

        /out은 인텔리제이로 빌드하였을 때 생기는 폴더이다.

        다음과 같이 설정해주면 인텔리제이에서도 /build 디렉토리를 사용하게 된다.

            allprojects {
             apply plugin: 'idea'
             idea {
               module {
                 outputDir file('build/classes/main')
                 testOutputDir file('build/classes/test')
               }
             }
             if(project.convention.findPlugin(JavaPluginConvention)) {
               // Change the output directory for the main and test source sets back to the old path
               sourceSets.main.output.classesDir = new File(buildDir, "classes/main")
               sourceSets.test.output.classesDir = new File(buildDir, "classes/test")
             }
            }

        굳이 이렇게 디렉토리를 나눈다고?

        - 참고 : [https://github.com/gradle/gradle/issues/2315](https://github.com/gradle/gradle/issues/2315)
        - 위의 이슈 때문에 굳이 나누게 되었다.

        /target은 maven으로 빌드하였을 때 나오는 폴더이다.

    자바에는 jar안에 들어있는 java 파일을 읽을 수 있는 표준적인 방법이 없다.

    따라서 옛날에는 'uber' jar를 사용했다.

    - 모든 클래스(의존성 및 애플리케이션)을 하나로 압축한다.
    - 뭐가 어디에서 온건지 알 수 없다.
        - 무슨 라이브러리를 쓰는지도 알 수 없다.
    - 내용은 다른데 이름이 같은 파일은 또 어떻게 처리할지에 대한 문제점도 있었다.

    따라서 스프링 부트는 전략을 바꿨다.

    - 내장 JAR: 기본적으로 자바에는 내장 JAR를 로딩하는 표준적인 방법이 없다.
        - Jar 파일안에 의존하고 있는 Jar 파일들을 다 그대로 넣어둔다.
    - 어플리케이션 클래스와 라이브러리의 위치를 구분한다.
    - org.springframework.boot.loader.jar.JarFile을 사용해서 내장 JAR를 읽는다.
        - 이 파일은 /build/app/org/springframework/boot/loader에 있다. 는 메이븐
        - gradle의 경우는 /build/libs에 어플리케이션을 통합한 jar 파일이 들어있다.
    - org.springframework.boot.loader.Launcher를 사용해서 실행한다.

    모든 jar 파일의 시작점은 `MANIFEST.INF`의  Main-Class 속성의 값이다. 이건 자바 스펙이다.