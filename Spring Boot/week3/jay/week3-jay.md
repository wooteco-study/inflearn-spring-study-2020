# 백기선 스프링부트 개념과 활용 1부

Created By: JunHo Park
Last Edited: Mar 09, 2020 10:49 AM

## 스프링 부트 프로젝트 생성기

skip

## 스프링 부트 프로젝트 구조

@SpringBootApplication은 패키지 루트에 위치해야 하며 여기서부터 component scan이 이루어지기 때문에 유의해야 한다.

## 의존성 관리 이해

메이븐 혹은 그래들로 의존성을 관리할 수 있다.

official 한 starter들은 spring-boot-starter-* 의 네이밍 규칙을 따른다.

### 메이븐, 그래들

Gradle은 Java 가상머신에서 실행되는 스크립트 언어이다.

규모가 커질수록 gradle을 사용하는 것이 체감상 유리하다.

출처: [https://okky.tistory.com/179](https://okky.tistory.com/179)

600명의 엔지니어가 1년동안 1분걸리는 빌드를 매주 42번 빌드를 진행할 때 들어가는 비용은

**600 engineers * $1.42/minutes * 42 builds/week * 44 work weeks/year = $1,600,000/year**

결과적으로 스크립트 길이나 가독성 면에서 Gradle이 앞서고 빌드와 테스트 실행결과가 더 빠르다. 스크립트 품질에서 앞선다.

출처: [https://bkim.tistory.com/13](https://bkim.tistory.com/13)

## 의존성 관리 응용

루트에서 관리하고 있는 친구들을 버전을 명시하지 않아도 된다.

## 자동 설정 이해

@EnableAutoConfiguration 과 @ComponentScan

ComponentScan이 먼저 읽히고 그 다음에 EnableAutoConfiguration 이 읽히게 된다.

    spring.factories

에서 미리 자동설정되는 클래스들을 볼 수 있다.

    @Target(ElementType.TYPE)
    @Retention(RetentionPolicy.RUNTIME)
    @Documented
    @Inherited
    @SpringBootConfiguration
    @EnableAutoConfiguration
    @ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
    		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
    public @interface SpringBootApplication {

기본적으로 SpringBootApplication에 EnableAutoConfiguration, ComponentScan이 들어있다.

## 자동 설정 만들기 1부: Starter와 AutoConfigure

@Conditional로 Auto-Configuration을 만들 수 있다.

스프링부트는 만들어진 jar에서  META-INF/spring.factories 파일의 존재를 체크한다.

- Auto-configurations must be loaded that way only. Make sure that they are defined in a specific package space and that they are never the target of component scanning. Furthermore, auto-configuration classes should not enable component scanning to find additional components. Specific @Imports should be used instead.

Auto-Configuration은 항상 저 방식으로만 로드되어야 한다. Component Scanning의 타겟이 되면 안된다. auto-configuration 클래스들은 다른 컴포넌트를 찾기 위해 추가적인 컴포넌트 스캐닝을 하면 안된다. 명시적인 @Import가 필요하다.

또 문서에서는 AutoConfigureAfter나 AutoConfigureBefore 애노테이션을 사용하여 순서를 정하라고 한다.

    @Configuration(proxyBeanMethods = false)
    @ConditionalOnClass({ JobLauncher.class, DataSource.class })
    @AutoConfigureAfter(HibernateJpaAutoConfiguration.class)
    @ConditionalOnBean(JobLauncher.class)
    @EnableConfigurationProperties(BatchProperties.class)
    @Import(BatchConfigurerConfiguration.class)
    public class BatchAutoConfiguration {

아무거나 찾아봤는데 예를들어, BatchAutoConfiguration같은 경우 HibernateJpaAutoConfiguration 이 있고 나서야 자동설정이 된다. 라고 써있다.

    @Configuration(proxyBeanMethods = false)
    @ConditionalOnClass({ LocalContainerEntityManagerFactoryBean.class, EntityManager.class, SessionImplementor.class })
    @EnableConfigurationProperties(JpaProperties.class)
    @AutoConfigureAfter({ DataSourceAutoConfiguration.class })
    @Import(HibernateJpaConfiguration.class)
    public class HibernateJpaAutoConfiguration {

    }

    @Configuration(proxyBeanMethods = false)
    @ConditionalOnClass({ DataSource.class, EmbeddedDatabaseType.class })
    @EnableConfigurationProperties(DataSourceProperties.class)
    @Import({ DataSourcePoolMetadataProvidersConfiguration.class, DataSourceInitializationConfiguration.class })
    public class DataSourceAutoConfiguration {

### Bean Conditions

    @Configuration(proxyBeanMethods = false)
    public class MyAutoConfiguration {

        @Bean
        @ConditionalOnMissingBean
        public MyService myService() { ... }

    }

이름을 명시하지 않으면 저렇게 MyService의 빈인것을 안다.

## 자동 설정 만들기 2부: @ConfigurationProperties

    @ConfigurationProperties(prefix = "spring.datasource")
    public class DataSourceProperties implements BeanClassLoaderAware, InitializingBean {

    	private ClassLoader classLoader;

    	/**
    	 * Name of the datasource. Default to "testdb" when using an embedded database.
    	 */
    	private String name;

우리가 흔히 썼던 application.properties에 썼던 저 데이터베이스 설정하는 부분을 찾아보았다.

이런식으로 미리 만들어놓고 라이브러리를 임포트하게 되면 빈을 재정의하는 수고를 덜 수 있다.

## 내장 웹 서버 이해

스프링부트는 웹 서버가 아니다

톰캣을 띄우는 과정없이 바로 실행해 주는게 스프링부트

    ServletWebServerFactoryAutConfiguration

## 내장 웹 서버 응용 1부 : 컨테이너와 포트

다른 서블릿 컨테이너 사용하는 방법

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <exclusions>
            <!-- Exclude the Tomcat dependency -->
            <exclusion>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-tomcat</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    <!-- Use Jetty instead -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-jetty</artifactId>
    </dependency>

Jetty, Undertow

Undertow가 더 빠르고 안정적이라고 한다.

출처: [https://okky.kr/article/543099](https://okky.kr/article/543099)

https 설정이 undertow가 조금더 간편하다.

출처: [https://www.hanumoka.net/2019/01/04/springBoot-20190104-springboot-undertow/](https://www.hanumoka.net/2019/01/04/springBoot-20190104-springboot-undertow/)

## 내장 웹 서버 응용 2부 : HTTPS와 HTTP2

## 독립적으로 실행 가능한 JAR

Spring maven plugin이 패키징을 한다. 패키징을 하면 Library들이 다 들어있다.

jar 파일을 unzip한다.

    unzip -q [name].jar

jar안에 jar파일들을 넣는다. 그래서 한방에 몰아서 넣는 예전 방식의 패키징방식보다 더 낫다.

JarLauncher가 이 안의 jar를 실행하게 된다.

## 스프링 부트 원리 정리

의존성관리, 자동설정, 내장 웹 서버
