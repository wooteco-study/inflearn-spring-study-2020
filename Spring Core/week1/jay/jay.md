# 백기선 스프링부트 1부

Created By: JunHo Park
Last Edited: Feb 16, 2020 11:38 PM

## IoC 컨테이너 1부

Inversion of Control: 의존관계 주입. 어떤 객체가 사용하는 의존객체를 직접 사용하는 것이 아니라 주입받아 사용하는 방법. 

스프링 초기에는 xml 을 이용하여 인젝션을 했지만 오늘 날에는 애노테이션 기반으로 사용하고 있다.

### Bean Factory

핵심이 되는 인터페이스 

- 공식문서

빈 팩토리는 스프링 빈 컨테이너를 접근하는 최상위 인터페이스이다. 이 인터페이스는 각각의 String 이름으로 고유한 많은 빈 정의들을 들고 있다. 이 팩토리는 프로토 타입의 빈 또는 싱클톤 타입의 빈을 리턴한다. 빈 팩토리 설정에 따라 어떤 타입의 빈이 리턴될지 결정된다. 스프링 2.0부터 다른 스코프들 (request, session)이 지원된다.

### 빈

Spring IOC 가 관리하는 객체

빈의 스코프: 싱글톤, 프로토타입 (매번 다른 객체)

의존성관리

라이프사이클 인터페이스

## IoC 컨테이너 2부: ApplicationContext와 다양한 빈 설정 방법

ComponentScan (basePackageClasses 를 쓰면 그 클래스가 위치한 곳 부터 컴포넌트 스캔을 진행한다)

## IoC 컨테이너 3부: @Autowire

autowired required false라는 옵션이 있으면 빈이 없으면 주입을 하지 않는다.

해당 타입의 빈을 전부 주입 받을 때는 List 로 autowired 를 하면 된다.

빈이 여러개 일 때 빈의 이름을 맞처주면 찾긴하는데 추천하지 않는다. 빈의 initilization 이후에 다른 라이프사이클 (BeanPostProcessor) 에서 Primary를 동작시킬 수 있다.

@Autowired 는 AutowireAnnotationBeanPostProcessor라는 클래스가 처리해준다.

## IoC 컨테이너 4부: @Component와 컴포넌트 스캔

Function을 사용한 빈 등록 (굳이 쓸 필요가 있는건가?)

## IoC 컨테이너 5부: 빈의 스코프

    @Scope(value="prototype", proxyMode=ScopedProxyMode.TARGET_CLASS)

싱글톤 빈이 프로토 타입의 빈을 직접 참조하면 안된다. 그렇기 때문에 프록시로 감싼다. 

넓은 범위의 라이프사이클(싱글톤)안에서 짧은 주기를 가져가는 빈을 쓸 때 참고.

### IoC 컨테이너 6부: Environment 1부. 프로파일

프로파일은 빈들의 그룹이며 환경별로 다른빈들을 쓸 수 있게 한다. 특정환경에서 특정 빈을 주입하는 경우도 마찬가지.

Environment 클래스

    environment.getActiveProfiles()

    @Configuration
    @Profile("test")
    public class ...

프로파일 조건 조합

    ! & | 
    @Profile("!prod & 어쩌구)

다음과 같이 할 수 있지만 그래도 최대한 간단하게 하는 것이 좋다.

## IoC 컨테이너 6부: Environment 2부. 프로퍼티

## IoC 컨테이너 7부: MessageSource

국제화 (i18n) 기능을 제공하는 인터페이스

## IoC 컨테이너 8부: ApplicationEventPublisher

옵저버 패턴의 구현체로 이벤트 기반의 프로그래밍을 할 때 유용하다.

이벤트 발행하는 기능은 ApplicationContext가 가지고 있다.

스프링 4.2부터는 이벤트를 만들 때 Event 관련 클래스를 상속받지 않아도 된다.

스프링은 비침투성 클래스를 만들기 위함(pojo)을 추구하는 방향성을 가지고 있기 때문에 그렇게 변경이 되었다.

이벤트를 수신하는 부분에서 @EventListener가 있어야 한다.

cf) 현재 스레드 찍기

    Thread.currentThread().toString()

이벤트 리스닝을 @Async로 비동기적으로 실행할 수 있다. 하지만 @Async만 붙인다고해서 다른 스레드에서 시작 되는 건 아니다. @EnableAsync 애노테이션을 @SpringBootApplication 쪽에 붙여준다. 그래야 제각각 스레드에서 돌게 된다.

## IoC 컨테이너 9부: ResourceLoader

## Resource 추상화

Resource 추상화의 특징은 java.net.URL 이라는 걸 래핑해서 만들었다. (클래스 패스를 가지고 가져오는 기능이 없었기도하다) 

스프링 내부에서 많이 사용하는 인터페이스

리소스 경로 없이 적었을 때는 ServletContextResource를 사용하고 classpath 접두어를 사용했을 시 ClassPathResource를 사용한다. 명시적으로 classpath를 쓰는 것을 추천.

내부모습

DefaultResourceLoader.class

    @Override
    	public Resource getResource(String location) {
    		Assert.notNull(location, "Location must not be null");
    
    		for (ProtocolResolver protocolResolver : getProtocolResolvers()) {
    			Resource resource = protocolResolver.resolve(location, this);
    			if (resource != null) {
    				return resource;
    			}
    		}
    
    		if (location.startsWith("/")) {
    			return getResourceByPath(location);
    		}
    		else if (location.startsWith(CLASSPATH_URL_PREFIX)) {
    			return new ClassPathResource(location.substring(CLASSPATH_URL_PREFIX.length()), getClassLoader());
    		}
    		else {
    			try {
    				// Try to parse the location as a URL...
    				URL url = new URL(location);
    				return (ResourceUtils.isFileURL(url) ? new FileUrlResource(url) : new UrlResource(url));
    			}
    			catch (MalformedURLException ex) {
    				// No URL -> resolve as resource path.
    				return getResourceByPath(location);
    			}
    		}
    	}

## Validation 추상화

애플리케이션에서 사용하는 객체 검증용 인터페이스이다. 어떤 계층과 관련이 없어서 모든 계층에서 사용가능하다.
