## 강의 듣고 정리하기
- [백기선 스프랑 코어](https://www.inflearn.com/course/spring-framework_core)

### 13. Resource(자원) 추상화
- 시작전
    - 생각보다 스프링은 빈 팩토리 이외의 기능을 많이 지원하고 있다 (단순히 빈 팩토리라고 하기에는)

- 추상화?!
    - java.net.URL 을 추상화 (이 친구가 무엇을 해주는 거지??)
        -
```java
public interface Resource extends InputStreamSource {
	boolean exists();
	boolean isReadable();
	boolean isOpen();
	URL getURL() throws IOException;
	URI getURI() throws IOException;
	File getFile() throws IOException;
	long contentLength() throws IOException;
	long lastModified() throws IOException;
	Resource createRelative(String relativePath) throws IOException;
	String getFilename();
	String getDescription();
}
```

- 리소스 읽어오기
    - Resource의 타입은 location 문자열과 __ApplicationContext 의 타입__ 에 따라 결정된다
        - ClassPathXmlApplicationContext -> ClassPathResource
        - FileSystemXmlApplicationContext -> FileSystemResource
        - WebApplicationContext -> ServletContextResource
    - 접두어를 쓰는 것을 추천해주신다
        - 코드만 봐서는 어디서 자원이 오는지 알기 어려움
        - 접두어를 쓰면 좀더 명시적
> 실제로 테스트코드를 짜보고 확인해보면 좋겠다 (접두어에 따라서 강제하기)

- 오홍...
    - 특정 인터페이스에 주입된 빈을 확인하기 위해서
        - getClass() 찍어보기
    - 실행해보기
        - classpath 지정안해주면?
            - WebApplicationContext 이기에.. ServletContextResource 사용(웹 어플리케이션 루트에서 상대 경로로 리소스를 찾음)
            - 내장형 톰캣에는 context path 가 지정되어있지 않음
            - 파일을 찾을 수 없음 
        - 먼가 프로젝트를 실행할 때... 내가 원하는 파일을 못찾으면.. 당황하지 말고... 이런 부분들을 확인해보자


- 자원은 무엇일까?
    - 스프링에서 xml 로 빈을 설정할때... 해당 xml 파일도 자원

- 응???? 어떻게 생각하면 되는거지??
- 자원을 가져오는 부분을 추상화하면 어떤 이점이 있을까?
    - InputStream 을 더 알게되면 좋을 것 같다
    - 무슨자원이든가에 상관없이 사용하는 방식을 추상화?

- new ClassPathXmlApplicationContext("xxx.xml")
    - 내부에선 ... resource.location 에 들어가게 됨

### 14. validation 추상화

14. validation 추상화 

- 객체 검증용 인터페이스
- 보통 웹에서 사용
    - 그래도 모든 계층이 사용할 수 있음

- Bean Validation 스팩..!

``` java
public interface Validator {
	boolean supports(Class<?> clazz);
	void validate(Object target, Errors errors);
}
```

### 15. 데이터 바인딩 추상화

15. 데이터 바인딩 추상화
- 기술적인 관점: 프로퍼티 값을 타겟 객체에 설정하는 기능
- 사용자 관점: 사용자 입력값을 애플리케이션 도메인 모델에 동적으로 변환해 넣어주는 기능
    - 보통 입력값은 문자열로 넣었을테니... 각 객체에 형에 맞게 변환해서 적용되도록

- PropertyEditor
    - 가장 고전적인 방식
    - 스레드-세이프 하지 않음
        - 내부에서 setValue, getValue 등을 할 때.. 상태가 공유될 수 있음
        - 빈으로 쓰지 말아용~
    - Object 와 string 간에 변환만 가능

- Converter
    - S 타입을 T 타입으로 변환할 수 있는 일반적인 변환기
    - 상태정보 없음 == 스레드 세이프
- Formatter
    - 스레드-세이프
        - 빈으로 등록 가능
            - 내부에 다른 빈도 주입 가능
``` java
@RestController
public class EventController {
    @GetMapping("/event/{id}")
    public String getEvent(@PathVariable Event event) {
        // ....
    }
}
```

- 등록된 포맷터 확인해보기
    - ConversionService 를 toString 으로 찍어보기

- 아... 그러면 코스때.. argumentResolver 같은 애들이...??
    - 그러면 어노테이션으로 사용자 입력을 구분하고
    - 해당 타입에 맞는 formatter 등을 찾아내서 변환?

### 17. SpEL (스프링 Expression Language)
- 문법
    - #{"표현식"}
        - 빈에 있는 애를 사용할수도 있음
    - ${"프로퍼티"}
    - [레퍼런스 참고](https://docs.spring.io/spring/docs/4.3.10.RELEASE/spring-framework-reference/html/expressions.html)

- 실제로 어디에서 쓰나?
    - @Value
    - @ConditionalOnExpression
    - 스프링 시큐리티
    - 스프링 데이터
    - Thymleaf


### Null-safety
> 스프링5 에 추가되었습니다

- 목적
    - (툴의 지원을 받아) 컴파일 시점에 최대한 NullPointerException 을 방지하는 것
        - intellij 설정 필요
            - @NonNull 등을 추가해주어야 함
