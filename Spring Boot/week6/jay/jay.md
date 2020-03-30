## 스프링 데이터 10부: MongoDB

몽고 DB는 JSON 기반의 도큐먼트 데이터베이스이다.

초창기에는 데이터가 날아간다 어쩐다 이슈가 있어서 사람들이 안쓴다고 뭐라 한적이 있었음.

Spring Data Mongo DB

테스트 할 때는 flapdoodle 에서 나온 embedded mongo db 의존성을 사용한다. 

## 스프링 데이터 11부: Neo4j

노드간의 연관 관계를 영속화하는데 유리한 그래프 데이터베이스이다. 친구의 친구의 친구의 관계를 표현하는데 유리하다.

기본 7474 포트

    @NodeEntity
    public class Account {}
    
    Session session = sessionFactory.openSession();

이 session에 엔티티를 save 한다.

    @RelationShip(type="has")
    Set<Role> roles;

## 스프링 데이터 12부: 정리

기본적으로 open session in view 가 true로 되어있다.

## 스프링 시큐리티 1부: Starter-Security

tip) mockMvc 테스트 할 때 print()를 먼저 찍으면 좋다. 

spring-security를 추가한다. 

일반 @GetMapping 으로 돌리면 401 Unauthorized 내려온다. 자동설정으로 모든 요청이 인증을 필요로 해진다. 

Basic Authentication, Form 검증

Accept 헤더에 따라 달라진다. 우리는 Accept 헤더를 보통 text/html 로 주로 보내왔다. 

    accept(MediaType.TEXT_HTML)

로 보낸다면 기본적으로 /login 으로 302 리다이렉션 으로 간다.

SecurityAutoConfiguration.class

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e38b106f-bbc8-432a-a858-6d5de216e184/_2020-03-29__5.27.09.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e38b106f-bbc8-432a-a858-6d5de216e184/_2020-03-29__5.27.09.png)

기본적으로 spring security에 있는 기본적인 클래스들을 상속받아서 만들어져있다. 

WebSecurityConfigurerAdapter.class

    protected final HttpSecurity getHttp() throws Exception {
    		if (http != null) {
    			return http;
    		}
    
    		DefaultAuthenticationEventPublisher eventPublisher = objectPostProcessor
    				.postProcess(new DefaultAuthenticationEventPublisher());
    		localConfigureAuthenticationBldr.authenticationEventPublisher(eventPublisher);
    
    		AuthenticationManager authenticationManager = authenticationManager();
    		authenticationBuilder.parentAuthenticationManager(authenticationManager);
    		authenticationBuilder.authenticationEventPublisher(eventPublisher);
    		Map<Class<?>, Object> sharedObjects = createSharedObjects();
    
    		http = new HttpSecurity(objectPostProcessor, authenticationBuilder,
    				sharedObjects);
    		if (!disableDefaults) {
    			// @formatter:off
    			http
    				.csrf().and()
    				.addFilter(new WebAsyncManagerIntegrationFilter())
    				.exceptionHandling().and()
    				.headers().and()
    				.sessionManagement().and()
    				.securityContext().and()
    				.requestCache().and()
    				.anonymous().and()
    				.servletApi().and()
    				.apply(new DefaultLoginPageConfigurer<>()).and()
    				.logout();
    			// @formatter:on
    			ClassLoader classLoader = this.context.getClassLoader();
    			List<AbstractHttpConfigurer> defaultHttpConfigurers =
    					SpringFactoriesLoader.loadFactories(AbstractHttpConfigurer.class, classLoader);
    
    			for (AbstractHttpConfigurer configurer : defaultHttpConfigurers) {
    				http.apply(configurer);
    			}
    		}
    		configure(http);
    		return http;
    	}

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/819b66b0-9e5e-4aec-a33f-e2e029accf55/_2020-03-29__5.30.20.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/819b66b0-9e5e-4aec-a33f-e2e029accf55/_2020-03-29__5.30.20.png)

UserDetailsServiceAutoConfiguration.class

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0cdfbecc-a425-4d25-be66-7dfa93e13fef/_2020-03-29__5.31.13.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0cdfbecc-a425-4d25-be66-7dfa93e13fef/_2020-03-29__5.31.13.png)

UserDetailsService라는게 없을 때 발동. 이 것을 쓰기 싫다면 우리가 UserDetailsService를 만들어야 한다.

테스트는 spring-security-test 의존성을 추가한다.

@WithMockUser 로 가짜 유저 정보 인증을 가정하고 테스트할 수 있다.

## 스프링 시큐리티 2부: 시큐리티 설정 커스터마이징

웹 시큐리티 설정

    @Configuration
    public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
       @Override
       protected void configure(HttpSecurity http) throws Exception {
           http.authorizeRequests()
                   .antMatchers("/", "/hello").permitAll()
                   .anyRequest().authenticated()
                   .and()
               .formLogin()
                   .and()
               .httpBasic();
       }
    }

UserDetailsService 타입의 빈이 등록이 되어야 스프링에서 기본적으로 만들어주는 디폴트 유저가 생성이 되지 않는다.

    loadUserByUserName()

이란 메서드를 오버라이드하여 사용자 정보를 핸들링한다. 

여기까지만 하면 PasswordEncoder 설정이 없어서 안된다.

PasswordEncoder의 설정

패스워드를 저장할 때 당연히 그냥 저장하면 안된다. 인코더를 사용하여 인코딩을 하여 저장해야 한다.

## 스프링 REST 클라이언트 1부: RestTemplate과 WebClient

RestTemplate, WebClient 두 가지 방식. 전자는 blocking, synchronous 후자는 non-blocking, asynchronous

RestTemplateBuilder를 주입받고 api를 두개 뚫고 하나는 5초, 하나는 3초를 지연시킨다.

    restTemplate.getForObject("url");
    
    restTemplate.getForObject("url");

blocking 이기 때문에 순차 실행이다.

WebClientBuilder를 주입받고 똑같이 실행한다.

    webClient.get().uri("url").retrieve().bodyToMono(String.class);

Mono타입을 리턴하기 때문에 구독을 해야 값이 반환된다.

결과적으로 5초가 걸린다. subscribe 할 때 실제 요청이 들어간다. 하지만 콜백 형태로 처리되기 때문에 다음 줄로 넘어간다.

## 스프링 REST 클라이언트 2부: 커스터마이징

baseUrl(): restTemplate의 baseUrl 세팅

RestTemplateCustomizer 로 HttpComponentClientHttpRequestFactory()를 넣어주면 java 의 http 를 쓰지않고 아파치 http client를 사용하게 된다. spring 의 psa

WebClient는 WebClientCustomizer bean을 만든다.
