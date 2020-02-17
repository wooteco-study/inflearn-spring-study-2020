# Section1 IoC 컨테이너와 빈

## IoC(Inversion of Control)란?

의존 관계 주입.

의존성을 개발자가 직접 생성해서 사용하는 게 아니라 생성자, 필드, setter등을 통해 주입 받아 사용하는 것을 의미한다.

아래와 같이 개발자가 직접 객체를 생성하지 않고

    public class BookSevice {
    
    	BookRepository bookRepository = new BookRepository();
    
    }

아래와 같이 작성해서 Container에 있는 Bean을 주입받아 사용하는 것을 말함.

    public class BookSevice {
    	
    	@Autowired
    	BookRepository bookRepository;
    
    }

 

Container에 들어 있는 bean을 주입받아 사용하는 것.

## BeanFactory

spring IoC container의 최상위 인터페이스

### Bean

**@Component**, @Repository, @Service, @Controller, @Configuration 등의 어노테이션이 있는 클래스가 Bean으로 등록된다. 

IoC 컨테이너가 관리하는 객체를 Bean이라고 한다.

Bean에 등록된 객체만 의존 주입을 받을 수 있다.

Bean에 등록된 객체는 singleton 스코프로 관리된다. (싱글 인스턴스)

라이프사이클 인터페이스. bean이 생성되기 전, 후 등 라이프사이클 각각의 순간에 실행 될 메서드를 구현할 수 있다. 

    public class BookService {
    	...
    
    	@PostConstruct
    	public void postConstruct() {
    		...
    	}
    
    }

### ApplicationContext

BeanFactory를 상속받는다.

Spring IoC 컨테이너는 Bean 설정파일이 있어야 한다. 

xml을 사용할 경우 application.xml에 Bean에 대한 설정을 해줘야 한다. 굳이 사용 예시를 보진 않겠다. 안 쓸 거니까.

Bean설정 파일을 Java 파일로 만든 게  Configuration.

    @Configuration
    public class ApplicationConfig {
    
    	
    	@Bean
    	public BookRepository bookRepository {
    		return new BookRepository();
    	}
    
    	@Bean
    	public BookService bookService() {
    		BookService bookService = new BookService();
    		bookService.serBookRepository(bookRepository());
    		return bookService;	
    	}
    	
    }

그러나 우리는 이렇게 사용하지 않는다.  스프링이 다 해준다.

@SpringBootApplicaiton → @ComponentScan

                                             → @SpringBootConfiguration → @Configuration

## @Autowired

생성자와 Field, Setter에 @Autowired를 설정할 수 있다. 

아래와 같이 주입 받을 수 있는 대상이 2개 이상인 경우. 우선순위나 사용 할 Bean을 설정해주지 않으면 애플리케이션 구동 당시에 에러가 난다. 

    @Service
    public class BookService {
    	@Autowired
    	BookRepository bookRepository;
    }
    
    @Repository
    public Interface BookRepository {
    }
    
    @Repository
    public class MybookRepository impplements BookRepository {
    }
    
    @Repository
    public class YourbookRepository impplements BookRepository {
    }

아래와 같이 @Primary를 달아주면 BookRepository가 주입받는 곳에서 @Primary가 달린 인스턴스를 찾아 주입해준다.

    @Service
    public class BookService {
    	@Autowired
    	BookRepository bookRepository;
    }
    
    @Repository
    public Interface BookRepository {
    }
    
    @@Repository @Primary
    public class MybookRepository impplements BookRepository {
    }
    
    @Repository
    public class YourbookRepository impplements BookRepository {
    }

혹은 @Autowired와 함께 @Qualifier("bean 이름")을 통해서 특정 bean을 주입받을 수 있다.

    @Service @Qualifier('myBookRepository")
    public class BookService {
    	@Autowired
    	BookRepository bookRepository;
    }
    
    @Repository
    public Interface BookRepository {
    }
    
    @@Repository @Primary
    public class MybookRepository impplements BookRepository {
    }
    
    @Repository
    public class YourbookRepository impplements BookRepository {
    }

그러나 @Primary가 더 타입 세이프하기 때문에 @Primary 사용을 추천.

BookRepository type의 모든 클래스를 주입받음.

    @Service @Qualifier('myBookRepository")
    public class BookService {
    	@Autowired
    	LiSt<BookRepository> bookRepositories;
    }
    
    @Repository
    public Interface BookRepository {
    }
    
    @@Repository
    public class MybookRepository impplements BookRepository {
    }
    
    @Repository
    public class YourbookRepository impplements BookRepository {
    }

@Autowired는 어떻게 적용되는 거지?

BeanFactory가 BeanPostProcessor를 찾음. BeanPostProcessor 타입 중 하나인 AutowiredAnnotationBeanPostProcessor를 다른 일반적인 bean에 적용해서 @Autowired를 적용시키는 것!

BeanPostProcessor란?

bean의 인스턴스를 만들고, bean initialization이라는 라이프사이클 이전과 이 후에 부가적인 작업을 할 수 있는데 그때 BeanPostProcessor가 실행됨

initialization이란?

인스턴스가 만들어진 후에 부가적인 작업을 하는 것. @PostConstruct

정리!

1. BeanFactory가 일반적인 bean을 비롯한 BeanPostProcessor를 찾아 등록함.
2. Bean이 만들어짐.
3. AutowiredAnnotationBeanPostProcessor가 실행됨(@Autowired가 적용됨)
4. initialization(@PostContruct)가 실행됨.

## Component와 Component scan

@SpringBootApplication에 @ComponentScan이 포함되어 있음. 

@ComponentScan 어노테이션에서 제일 중요한 부분은 basePackages와 basePackageClasses이다.

basePackages는 문자열, basePackageClasses는 클래스.

각 값에 전달된 클래스를 기준으로 component를 scan한다.

@SpringBootApplication이 달린 클래스가 포함된 패키지를 기준으로 해당 패키지에 포함된 모든 클래스가 스캔의 대상이 된다.

excludeFilters 속성을 통해서 scan 대상을 제외할 수 있다.

![Section1%20IoC/_2020-02-16__9.24.09.png](Section1%20IoC/_2020-02-16__9.24.09.png)

ConfigurationClassPostProcessor라는 BeanFactoryPostProcessor에 의해 scan이 동작함.

다른 Bean들을 만들기 전에 적용을 함. 다른 bean이 등록되기 전에 동작해서 bean을 스캔하는 것.

ComponentScan에서 중요한 것!

컴포넌트 스캔의 역할

컴포넌트 스캔의 애노테이션 속성의 역할(Filter)

컴포넌트 스캔의 대상

## 빈의 스코프

스프링에서는 아무 설정을 하지 않으면 Bean이 싱글톤 스코프로 생성되어 사용된다. 

스코프 타입

- 싱글톤 : 애플리케이션 전반에 걸 쳐 하나의 인스턴스만 존재함.
- 프로토타입 : 매번 새로운 인스턴스를 만들어 사용함.
    - @scope("prototype") 과 같이 어노테이션을 이용해 설정할 수 있음.
    - Request
    - Session
    - WebSocket
    - Thread

    주의

    프로토타입 빈이 싱글톤 빈을 참조하는 것은 아무 문제가 없음.

    그러나 싱글톤타입 빈이 프로토타입 빈을 참조하면 싱글톤타입이 참조하고 있는 프로토타입의 빈은 업데이트되지 않는다. 계속 같은 빈이 참조된다. 바뀌지 않는다! 프로토타입을 사용하는 건 빈을 사용할 때마다 다른 빈을 사용하겠다는 것인데 바뀌지 않는다는 것이다.

    해결 방법은?

    @Scope(proxyMode = ScopedProxyMode.TARGET_CLASS) 와 같이 어노테이션을 이용해 해결할 수 있다. 이렇게 어노테이션을 하면 프로토타입 빈으로 설정된 빈을 Proxy로 감싸서 Proxy를 참조시키게 하는 것. 프록시 빈이 프로토 타입을 상속해서 만들었기 때문에 타입은 프로토와 같다. 

    ![Section1%20IoC/_2020-02-17__9.31.55.png](Section1%20IoC/_2020-02-17__9.31.55.png)

    그런데 어차피, 프로토타입을 사용할 상황은 거의 없음.

    ## ApplicationContext : Environment - profile

    ApplicationContext는 BeanFactory의 역할만 하는 것은 아니다.

    ApplicationContext가 상속하고 있는 EnvironmentCapable, EnvironmentCapable이 제공하는 기능 중 하나인 Profile 기능에 대해서 알아보자.

    아래와 같이 프로파일에 따라 어떻게 bean을 등록할 지 설정할 수 있다.

        @Configuration
        @Profile("test")
        public class TestConfiguration  {
        
        	@Bean
        	public BookRepository bookRepository() {
        		return new TestBookREpository();
        	}
        }

    아래와 같이 bean마다 profile을 설정할 수도 있다.

        @Repository
        @Profile("test")
        public class TestBookRepository implements BookRepository {
        }
        
        
        @Profile("!prod")
        // prod profile이 아닌 경우에만 빈 등록을 하라는 것

    EditConfiguration의 activeProfile에 어떤 profile을 사용할지 설정할 수 있다.

    ## ApplicationContext : Environment - property

    EditConfiguration의 VM Options에 property를 key-value로 줄 수 있다.

    ![Section1%20IoC/_2020-02-17__9.54.33.png](Section1%20IoC/_2020-02-17__9.54.33.png)

        Environmen encvironment = new ApplicationContext();
        encironment.getProperty("app.name"); // spring5

    application.properties에도 key-value 형태로 property를 줄 수 있다.

        application.properties
        
        app.name=spring

        @SpringBootApplication
        @PropertySource("classpath:/app.properties")
        public class Spring5Application {
        
        	public static void main(String[] args) {
        		SpringApplication.run(Spring5Application.class, args);
        	}
        }

    VM Option과 application.properties에 동일한 key값으로 다른 value가 설정되어 있으면 어디에서 더 우선순위가 높을까? VM Option의 우선순위가 더 높아서 같은 키에 한해서는 applicaiton.properteis의 값이 무시됨.

    ## ApplicationContext :  MessageSource

    국제화 기능(i18n)을 제공하는 인터페이스.

        messages.properteis
        
        greeting=Hello {0}

        messages_ko_KR.properties
        
        greeting=안녕, {0}

        @Autowired
        MessageSource messageSource;
        
        messageSource.getMessage("greeting", new String[]{"soojin"}, Locale.KOREA) // 안녕 soojin
        messageSource.getMessage("greeting", new String[]{"soojin"}, Locale.DEFAULT) // Hello soojin

## ApplicationContext : ApplicationEventPublisher

옵저버 패턴의 구현체로 이벤트 기반의 프로그래밍을 할 때 유용한 인터페이스라고 한다. 

이벤트는 빈으로 등록되는 게 아니라, 원하는 데이터를 담아 전송할 수 있다.

스프링 4.2 이하 버전에서 사용하는 방법

    public class MyEvent extends ApplicationEvent {
    
    	private int data;
    
    	public MyEvent(Object source) {
    		super(source);
    	}
    
    	public int getData() {
    		return data;
    	}
    } 

    @Component
    public class MyEventHandler implements ApplicationListener<MyEvent> {
    
    	@Override
    	public void onApplicationEvent(MyEvent event) {
    		System.out.println("이벤트 받았다. 이벤트는" + event.getData());
    	}
    
    }

    @Autowired
    ApplicationEventPublisher publishEvent;
    
    publishEvent.(new MyEvent(this, 100));

스프링 4.2 이상에서 사용하는 방식

이벤트는 bean이 아니지만 handler는 빈이어야 한다.

핸들러는 1개 이상 만들 수 있고 순서는 보장되지 않지만 둘 다 실행된다. 

    public class MyEvent {
    
    	private int data;
    	private Object source;
    
    	public MyEvent(Object source, int data) {
    		this.source = source;
    		this.data = data;
    	}
    
    	public Object getSource() {
    		return source;
    	{
    
    	public int getData() {
    		return data;
    	}
    } 

    @Component 
    public class MyEventHandler {
    	@EventListener
    	@Order(Ordered.HIGHEST_PRECEDENCE) // 1순위. 우선순위 주는 어노테이션
    	public void handle(MyEvent event) {
    		System.out.println("이벤트 받았다. 이벤트는" + event.getData());
    	}
    
    }

    @Component 
    public class AnotherHandler {
    	@EventListener
      @Order(Ordered.HIGHEST_PRECEDENCE + 2) // 2순위 순서
    	public void handle(MyEvent event) {
    		System.out.println("어나더 핸드러 이벤트 받았다. 이벤트는" + event.getData());
    	}
    
    }

    @Autowired
    ApplicationEventPublisher publishEvent;
    
    publishEvent.(new MyEvent(this, 100));

## ApplicationContext : ResourceLoader

리소스를 읽어오는 기능을 하는 인터페이스이다!

    @Component
    public class AppRunner implements ApplicationRunner {
    
    	@Autowired
    	ResourceLoader resourceLoader;
    
    	@Override
    	public void run(ApplicationArguments args) throws Exception {
    		resourceLoader.getResource("classpath:test.txt");
    	}
    }