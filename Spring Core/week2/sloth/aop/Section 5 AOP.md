# Section 5 AOP

## 스프링 AOP

흩어진 Aspect를 모듈화할 수 있는 프로그래밍 기법

![Section%205%20AOP/_2020-02-23__8.06.40.png](Section%205%20AOP/_2020-02-23__8.06.40.png)

### AOP 주요 개념

Aspect : 흩어진 관심사를 모듈화 한 것

Target : 적용될 대상

Advice : 해야할 일

JoinPoint : Aspect가 적용되는 시점

PointCut : 어디에 적용되어야 하는지

AOP 적용 방법

- 컴파일 타임 : 자바 파일을 class파일로 만들 때 aop를 적용해서 바이트 코드를 만들어 냄.
- 로드 타임 : 컴파일 한 뒤, 클래스 파일을 로딩하는 시점에 클래스 정보를 변경한다. load time weaving. 자바 에이전트 설정이 필요하다.
- 런타임 : A라는 타입의 Bean을 만들 때 Proxy Bean을 만들어서 이 Proxy bean이 A가 가지고 있는 메서드를 호출하기 전에 AOP에서 정의한 기능을 수행하고  의도된 메서드를 호출한다. 즉, 프록시 Bean을 만들어서 AOP를 적용한다.

## 프록시 기반 AOP

![Section%205%20AOP/_2020-02-23__8.33.17.png](Section%205%20AOP/_2020-02-23__8.33.17.png)

- 프록시 패턴을 사용하면 기존 코드의 변경 없이 접근 제어와 부가 기능 추가 등의 작업을 할 수 있다.
- 클라이언트는 인터페이스 타입을 통해서 프록시 객체를 사용한다.
- 프록시는 원래 사용하려고 한 객체를 가지고 있다.

## Spring AOP

spring-boot-starter-aop 의존성 추가하는 것으로 사용 가능.

### execution 기반

    @Component
    @Aspect
    public class PerfAspect {
    		@Around("execution(*.me.whiteship..*.EventService.*.(..))") //포인트 컷
    	// 해야할 일               실행하게 될 메서드
    	public Object logPerf(ProceedingJoinPoint pjp) throws Throwable {
    		long begin = System.currentTimeMills();	
    		Object retVal = pjo.proceed();
    		System.out.println(System.currentTimeMills() = begin);
    		return retVal;
    	}
    
    }

### Annotation 기반

    @Retention(RetentionPolicy.CLASS) // 애노테이션 정보를 어디까지 유지할 것인가, SOURCE나 RUNTIME으로 하지 말고 기본값으로 사용 추천
    @Target(ElementType.METHOD)
    public @interface PerfLogging { 
    }
    
    @Component
    @Aspect
    public class PerfAspect {
    		@Around("@annotation(PerfLogging)")
    	public Object logPerf(ProceedingJoinPoint pjp) throws Throwable {
    		long begin = System.currentTimeMills();	
    		Object retVal = pjo.proceed();
    		System.out.println(System.currentTimeMills() = begin);
    		return retVal;
    	}
    
    
    	// bean으로 등록된 simpleEventService의 모든 메서드가 실행되기 전에 hello를 찍음 
    	@Before("bean(simpleEventService)")
    	public void hello() {
    		System.out.println("hello");
    	}
    
    }