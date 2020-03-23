# 백기선 스프링부트 개념과 활용 (5주차)

Created: Mar 22, 2020 9:05 AM
Lesson Date: Mar 23, 2020
Status: In Progress
Type: Spring

## 스프링 웹 MVC 4부: 정적 리소스 지원

ResourceHttpRequestHandler: 정적 리소스를 처리하는 작업을 한다.

Last Modified 헤더를 보고 304 로 내려 주어 불필요한 오버 헤드를 줄인다.

ResourceHttpRequestHandler 클래스 찾아보기

    @Override
    	public void handleRequest(HttpServletRequest request, HttpServletResponse response)
    			throws ServletException, IOException {
    
    		// For very general mappings (e.g. "/") we need to check 404 first
    		Resource resource = getResource(request);
    
    // 중략....

getResource 로 자원을 가져온다.

    @Nullable
    	protected Resource getResource(HttpServletRequest request) throws IOException {
    		String path = (String) request.getAttribute(HandlerMapping.PATH_WITHIN_HANDLER_MAPPING_ATTRIBUTE);
    		if (path == null) {
    			throw new IllegalStateException("Required request attribute '" +
    					HandlerMapping.PATH_WITHIN_HANDLER_MAPPING_ATTRIBUTE + "' is not set");
    		}
    
    		path = processPath(path);
    		if (!StringUtils.hasText(path) || isInvalidPath(path)) {
    			return null;
    		}
    		if (isInvalidEncodedPath(path)) {
    			return null;
    		}
    
    		Assert.notNull(this.resolverChain, "ResourceResolverChain not initialized.");
    		Assert.notNull(this.transformerChain, "ResourceTransformerChain not initialized.");
    
    		Resource resource = this.resolverChain.resolveResource(request, path, getLocations());
    		if (resource != null) {
    			resource = this.transformerChain.transform(request, resource);
    		}
    		return resource;
    	}

해당자원에 대해서 Modified 된지를 체크한다.

    // Header phase
    if (new ServletWebRequest(request, response).checkNotModified(resource.lastModified())) {
    	logger.trace("Resource not modified");
    	return;
    }

Resource는 스프링 코어에 있는 인터페이스인데 거기에 정의된 lastModified 필드를 보고 수정 된지를 판단한다.

## 스프링 웹 MVC 5부: 웹JAR

웹JAR 맵핑  /webjars/**

버전 생략하고 사용하려면 webjars-locator-core 의존성 추가

## 스프링 웹 MVC 6부: index 페이지와 파비콘

index 페이지, favicon 만들기...

## 스프링 웹 MVC 7부: Thymeleaf

스프링부트가 자동설정지원하는 템플릿 엔진

FreeMarker, Groovy, Thmeleaf, Mustache

### JSP를 권장하지 않는 이유

스프링부트에서 지향하는 바와 맞지 않는다. 스프링부트는 독립적으로 실행가능한 임베디드 톰캣으로 배포를 지향한다. JSP를 사용하면 jar 패키징할 때 쓸 수 없고 war 패키징을 써야 한다. Undertow는 jsp를 지원하지 않는다.

(테스트도 쭉 만드심... Model로 화면에 내림)

타임리프를 쓰기때문에 렌더링 (Content Body를 확인하는 과정) 이 된다. 예전에 JSP를 썼을 때 렌더링 결과를 알기가 힘들었다. 렌더링 자체를 서블릿 엔진이 해야 한다. 

- 타임리프는 서블릿 컨테이너와 별개로 동작하기 때문에 응답 바디를 알 수 있다. (MockMvc는 실제 서블릿 컨테이너를 띄우는게 아니기 때문에 실제 서블릿 컨테이너가 하는일을 다 하지 못한다)

## 스프링 웹 MVC 8부: HtmlUnit

html을 단위테스트 하기 위한 툴이다.

- htmlunit-driver, htmlunit 을 테스트 스코프로 의존성을 추가한다.
- WebClient를 이용해서 테스트한다.
- 화면의 form 요청도 검증할 수 있다.
- 특정 브라우저로 테스트할 수 있다.

## 스프링 웹 MVC 9부: ExceptionHandler

Whitelabel 페이지는 기본적인 에러핸들러가 처리한다.

curl (머신클라이언트)로 요청하면 json으로 응답이온다.

    @RequestMapping(produces = MediaType.TEXT_HTML_VALUE)
    	public ModelAndView errorHtml(HttpServletRequest request, HttpServletResponse response) {
    		HttpStatus status = getStatus(request);
    		Map<String, Object> model = Collections
    				.unmodifiableMap(getErrorAttributes(request, isIncludeStackTrace(request, MediaType.TEXT_HTML)));
    		response.setStatus(status.value());
    		ModelAndView modelAndView = resolveErrorView(request, response, status, model);
    		return (modelAndView != null) ? modelAndView : new ModelAndView("error", model);
    	}
    
    @RequestMapping
    public ResponseEntity<Map<String, Object>> error(HttpServletRequest request) {
    	HttpStatus status = getStatus(request);
    	if (status == HttpStatus.NO_CONTENT) {
    		return new ResponseEntity<>(status);
    	}
    	Map<String, Object> body = getErrorAttributes(request, isIncludeStackTrace(request, MediaType.ALL));
    	return new ResponseEntity<>(body, status);
    }

### @ExceptionHandler

- @Controller 내부에서 사용하면 해당 컨트롤러에서만의 예외를 처리한다. 전역적으로 사용하려면 @ControllerAdvice 사용
- 기본적인 error path는 /error 폴더의 404.html, 5xx.html 같은 파일을 만들면 된다.

## 스프링 웹 MVC 10부: Spring HATEOAS

Rest api 를 만들 때 리소스와 연결되어있는 리소스 정보를 같이 제공하고 클라이언트는 이 정보를 활용한다.

SpringWeb만 추가하면 ObjectMapper가 자동으로 들어있다. (참고로 깃헙 api는 이렇게 구성되어있다)

## 스프링 웹 MVC 11부: CORS

Single-Origin Policy는 같은 origin에만 요청을 보낼 수 있다.

Cross-Origin Resource Sharing은 서로 다른 origin 에서 share 할 수 있다.

Origin 이란 uri 스키마 (http, https) + hostname + 포트 를 의미한다.

SOP는 예를들어, [http://localhost:8080](http://localhost:8080) 는 localhost:18080에 있는 리소스에 접근할 수 없다는 것.

    @CrossOrigin(origins = "http://localhost:18080")

여러 컨트롤러에 걸쳐서 사용하려면 WebMvcConfigurer를 구현한 클래스를 만들어서 addCorsMapping 으로 전역 설정을 한다.

## 스프링 데이터 2부: 인메모리 데이터베이스

스프링부트가 지원해주는 인 메모리 데이터베이스 (H2, HSQL, Derby)

spring-jdbc 가 존재하며 datasource 와 jdbcTemplate 같은 빈들을 설정해 준다.

그리고 아무것도 설정하지 않은 상태로 앱을 띄우면 자동으로 인메모리 데이터베이스를 띄워준다.

(h2 콘솔 띄워서 실습...)

## 스프링 데이터 3부: MySQL

DBCP는 DB 커넥션을 미리 만들어 놓는 작업을 한다. DBCP는 애플리케이션 성능에 아주 핵심적인 역할을 한다.

HikariCP

- autoCommit true: SQL을 실행할 때 마다 commit 명시를 하지 않아도 commit이 된다.
- connectionTimeout: DBCP 풀에서 커넥션을 앱으로 어느기간동안 전달을 하지 못하면 에러를 낼 것인지 설정. 디폴트 30초
- maximumPoolSize: 커넥션 객체의 개수 설정. 커넥션 개수가 많다고 해서 다 실행할 수있는 것은 아님. 동시에 일할 수 있는 커넥션 개수는 cpu core 개수와 같다. 그 이상의 커넥션은 대기한다.

    spring.datasource.hikira.*

(갑자기 도커)

MySQL은 상용애플리케이션에  쓰려면 돈을 내야한다. GPL이기 때문에 소스 공개의 의무가 있다. 

MySQL대신 Maria DB를 사용할 수 있다. 커뮤니티 버전의 MySQL. 라이센스는 필요없지만 GPL.

## 스프링 데이터 4부: PostgreSQL

위와 거의 같음.

## 스프링 데이터 5부: 스프링 데이터 JPA 소개

spring-boot-starter-data-jpa 의존성 추가

- JPA: ORM을 위한 자바 표준
- Hibernate의 모든 기능을 JPA가 다 커버하진 않는다.
- Spring Data JPA는 JPA 표준 스펙을 아주 쉽게 추상화 시킨것.
- EnableJpaRepository는 자동설정 됨.

## 스프링 데이터 6부: 스프링 데이터 JPA 연동

@DataJpaTest: 슬라이싱 테스트 할 때 사용 → DataSource, JdbcTemplate과 해당 엔티티의 repository를 주입받을 수 있다.

- 슬라이싱 테스트는 인메모리 데이터베이스가 필요하다.
- SpringBootTest를 사용하면 application.properties적용되어 인메모리를 사용하지 않는다. 하지만 테스트를 돌릴 때는 임베디드 디비를 사용하는게 낫다.

## 스프링 데이터 7부: 데이터베이스 초기화

    spring.jpa.hibernate.ddl-auto=

create-drop: 처음에 만들고 앱 종료할 때 지운다.

create: 초반에 띄울 때 지운다.

update: 기존 데이터를 유지한다.

validate: 운영에서는 밸리데이션 사용. 제대로매핑이 되는지 확인만 한다. 그리고 아래의 generate-ddl을 false로 준다.

    spring.jpa.generate-ddl=true

schema.sql에 초기 데이터베이스 설정을 넣을 수 있다. 

schema-${platform}.sql 네이밍을 통해 해당 플랫폼에서 동작하는 sql문을 만들 수 있다.

## 스프링 데이터 8부: 데이터베이스 마이그레이션

Flyway 는 데이터베이스 버전 관리를 한다.

### 마이그레이션 디렉토리

db/migration 혹은 db/migration/{vendor}

spring.flyway.locations로 변경 가능

### 마이그레이션 파일 이름

V숫자__이름.sql (언더바 두개고 이름은 서술적으로)

한 번 적용된 script 파일을 절대 건드리지 말고 새로운 파일을 만들어야 한다.
