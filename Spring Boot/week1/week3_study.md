# week3_study

> 강의문서 외, 스터디 중 추가로 논의한 내용

### Maven vs **Gradle**

Gradle은 Java 가상머신에서 실행되는 스크립트 언어이다.

규모가 커질수록 gradle을 사용하는 것이 체감상 유리하다.

출처: [https://okky.tistory.com/179](https://okky.tistory.com/179)

600명의 엔지니어가 1년동안 1분걸리는 빌드를 매주 42번 빌드를 진행할 때 들어가는 비용은

**600 engineers * $1.42/minutes * 42 builds/week * 44 work weeks/year = $1,600,000/year**

결과적으로 스크립트 길이나 가독성 면에서 Gradle이 앞서고 빌드와 테스트 실행결과가 더 빠르다. 스크립트 품질에서 앞선다.

출처: [https://bkim.tistory.com/13](https://bkim.tistory.com/13)

### Build Directory

`/target` : maven으로 빌드할때 생기는 폴더

`/build` : gradle로 빌드할때 생기는 폴더

`/out` : 인텔리제이로 빌드할때 생기는 폴더

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

굳이 이렇게 디렉토리를 나누는 이유는 [https://github.com/gradle/gradle/issues/2315](https://github.com/gradle/gradle/issues/2315)

### Model Mapper

- 다들 많이 쓰진 않는 분위기
- 슬로스는 orika-mapper 라는 것도 쓴다더라

### 우리가 스프링부트를 NONE으로 서버모드를 끄고 사용하는 경우가 있을까?에 대한 논의

### groupId, artifactId

    groupId : 패키지 명을 입력
    
    - groupId는 당신의 프로젝트를 모든 프로젝트 사이에서 고유하게 식별하게 해 주는 것이다.
    - 따라서, groupId에는 네이밍 스키마를 적용하도록 한다.
        - groupId는 package 명명 규칙을 따르도록 한다.
        - 즉, 최소한 당신이 컨트롤하는 도메인 네임이어야 한다.
        - 하위 그룹은 얼마든지 추가할 수 있다.
        - 예: `org.apache.maven`, `org.apache.commons`
    
    artifactId : 원하는 이름을 주면 됨
    
    - artifactId는 버전 정보를 생략한 `jar` 파일의 이름이다.
        - 이름은 원하는 것으로 아무거나 정해도 괜찮다.
        - 단, 소문자로만 작성하도록 한다.
        - 단, 특수문자는 사용하지 않는다.
    - 만약 써드 파티 `jar` 파일이라면, 할당된 이름을 사용해야 한다.
        - 예: `maven`, `commons-math`
    
    참고 : https://johngrib.github.io/wiki/groupId-artifactId