# week1-study

> 강의문서 외, 스터디 중 추가로 논의한 내용

- maven의 `/target` → gradle의 `/build`
- `Resource`와 `classpath, file`같은 접두사를 이용하자

### `ApplicationEventPublisher`

- 스프링의 `ApplicationEventPublisher`와 `@EventListener`를 이용해 이벤트를 쏘는 기능을 구현할 수 있다.
- 이점
    - 서비스간의 불필요한 참조(의존성)를 줄일 수 있다.
    - 메소드를 간단하게 유지할 수 있다.
- `@TransactionalEventListener`
    - `@EventListener` 는 트랜잭션 범위 내에서 동기적으로 실행된다. 반면 `@TransactionalEventListener`는 커밋 전/후 등을 자유롭게 지정할 수 있다.
    - 즉, 설정에 따라 **이벤트 핸들러 호출이 커밋 전, 후 등에 됨을 보장받을 수 있다.**
    - 예시
        - DB save 작업 후 이벤트 핸들러를 호출하는 로직이 있다. 이벤트 핸들러에선 다른 서버로 save 됬음을 알리는 콜을 쏜다.
        - 일반적인 로직이라면 rollback이 가능하므로, 서비스단의 `@Transactional` 만으로도 트랜잭션을 보장할 수 있다. 하지만 위와 같은 경우, 이미 보내서 취소할 수 없다.
        - 이 경우, `@TransactionalEventListener` 를 이용해, 이전의 DB save가 커밋 된 후에 이벤트 핸들러가 호출되도록 보장할 수 있다.

### Bean Life Cycle → 다음주 논의

- 참고자료 : [https://www.slipp.net/wiki/display/SLS/Spring+Core](https://www.slipp.net/wiki/display/SLS/Spring+Core)