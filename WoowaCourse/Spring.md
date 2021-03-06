# 스프링 부트 프로젝트 생성하기

1. spring initializer
2. java project 생성 후 의존성 추가
3. 온라인 initializer

[Spring Initializr](https://start.spring.io/)

# 의존성 관리 이해

'io.spring.dependency-management' 가 Spring 의존성 버전 관리를 해준다.

```
plugins {
    id 'org.springframework.boot' version '2.2.2.RELEASE'
    **id 'io.spring.dependency-management' version '1.0.8.RELEASE'
		// cf. maven에서는 parent**
    id 'java'
}
.
.
.
dependencies {
    **implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
		// spring.dependency-management 덕분에 version을 명시하지 않고 사용할 수 있다.
		// 이 의존성은 또 다른 라이브러리들을 추가해준다.
}
```

1. 'io.spring.dependency-management'에는 각 의존성에 따른 버젼을 설정해두었다.
2. 사용자는 version을 명시하지 않아도 의존성을 추가할 수 있다.

    하지만 version을 명시하는 것이 좋다. (server에 배포할 때 다른 version이 사용될 수 있기 때문에)

3. Spring boot에서 의존성 관리를 해주면서 개발자는 관리할 의존성이 별로 없어졌다. 

---

```java
repositories {
    mavenCentral()
    jcenter()
}
```

`repositories` 는 각종 의존성(라이브러리)들을 어떤 원격 저장소에서 받을지를 정합니다. 기본적으로 mavenCentral을 많이 사용하지만, 최근에는 **라이브러리 업로드 난이도** 때문에 jcenter도 많이 사용합니다.

mavenCentral은 이전부터 많이 사용하는 저장소지만, 본인이 만든 라이브러리를 업로드하기 위해 많은 과정과 설정이 필요하다보니 개발자들이 직접 만든 라이브러리를 업로드하는 것이 힘들어 점점 공유가 안되는 상황이 발생했습니다.

이런 문제점을 개선하여 최근에 나온 것이 jcenter입니다. 그리고 jcenter에 라이브러리를 업로드하면 mavenCentral에도 업로드될 수 있도록 자동화를 할 수 있습니다. 여기서는 mavenCentral, jcenter 둘 다 등록해서 사용하겠습니다.

---

## 자동 설정 이해

@SpringBootApplication  = @SpringBootConfiguration + @ComponentScan + @EnableAutoConfiguration

Bean은 사실 두 단계로 나눠서 읽힘

- 1단계: @ComponentScan
- 2단계: @EnableAutoConfiguration

### ComponentScan

1. @Component 을 scan해준다.

    Component: @Configuration @Repository @Service @Controller @RestController

2. Sacn 받지 않도록 설정해준 Component는 제외하고 Scan한다.

```java
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
```

3. ComponentScan을 시작하는 곳 하위 package만 Scan하는 것 주의할 것

### EnableAutoConfiguration

- @EnableAutoConfiguration에 등록된 Annotation(@XXXConfiguration)이 있다면 Bean 등록해준다.
    - gradle 설정을 통해 EnableAutoConfiguation 라이브러리를 받았다. (external lib)
    - 그 라이브러리의 meta file인 spring.factories에, @EnableAutoConfiguration을 통해 등록할 annotation 목록이 있다.
    - @XXXConfiguartion 역시 @Configuration이다.
    - 조건이 있어 조건에 따라 Bean 등록을 하게 된다.
---
### 내장 웹 서버 이해
부트는 서버가 아니다.
1. 톰캣 객체 생성
2. 포트 설정
3. 톰캣에 컨텍스트 추가
4. 서블릿 만들기
5. 톰캣에 서블릿 추가
6. 컨텍스트에 서블릿 맵핑
7. 톰캣 실행 및 대기

이 모든 과정을 보다 상세히 또 유연하게 설정하고 실행해주는게 바로 스프링 부트의 자동 설정

* ServletWebServerFactoryAutoConfiguration(서블릿 웹 서버 생성)
    * TomcatServletWebServerFactoryCustomizer(서버 커스터마이징)
* DispatcherServletAutoConfiguration
    * 서블릿 만들고 등록
---
## HTTPS, HTTP2
### HTTPS
(1) Key store 생성
```
$ keytool -genkey -alias tomcat -storetype PKCS12 -keyalg RSA -keysize 2048 -keystore keystore.p12 -validity 4000
application.yml
```
명령어 수행한 위치에 키스토어가 생성된다. 
(2) Key 등록
```
[application.yml]
server:
ssl:
  key-store: keystore.p12
  key-store-type: PKCS12
  key-store-password: 123456
  key-alias: tomcat
```
이렇게 SSL 키를 등록하고 스프링부트 애플리케이션을 실행하면, localhost:8080으로 접근이 불가하다. 앞으로 애플리케이션으로의 모든 접근은 https로 해야한다.

(3)
추가적으로 http 접근도 가능하게 설정하려면 아래와 같이 애플리케이션 코드에 http 요청을 받기 위한 커넥터를 추가해주면 된다. 대신 application.yml에서 https의 포트를 변경해준다.

```
 @Bean
   public ServletWebServerFactory serverFactory() {
       TomcatServletWebServerFactory tomcat = new TomcatServletWebServerFactory();
       tomcat.addAdditionalTomcatConnectors(createStandardConnector());
       return tomcat;
  }
​
   private Connector createStandardConnector() {
       Connector connector = new Connector("org.apache.coyote.http11.Http11NioProtocol");
       connector.setPort(8080);
       return connector;
  }
```
---
### 독립적으로 실행 가능한 JAR
**의존성 packing을 통해 받은 외부 라이브러리를 통해 JAR 파일 하나로 독립적으로 실행 할 수 있다.**

- gradle run (run하기 위해 필요한 파일 생성, jar 생성X )
- gradle boot jar (jar 생성)
- build (run + gardle boot jar에 필요한 파일 생성)
    - gradle boot jar 명령어를 통해 jar 파일을 만드려면 gradle에 main class 명시하고, plugin에 application 추가해주어야 함
    - gradle :  mf파일을 통해 실행한다.
    - jar : arFile, Launcher를 사용해서 실행한다.
- jar 안에 외부 라이브러리 jar로 가지고 있다.

### SpringApplication 실행

다음과 같은 방법으로 SpringApplication의 설정을 custom하여 실행할 수 있다.

```java
// 방법 1
SpringApplication app = new SpringApplicaiton(SpringInitApplication.class);
app.run(args);

// 방법 2
new SpringAppplicationBuilder().sources(SpringInitApplication.class).run(args);
```

----

### Event Listener

https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-spring-application.html#boot-features-application-events-and-listeners



Spring Container가 실행 된 후 Event Listener

```JAVA
public class Listener implements ApplicationListener<ApplicationStartedEvent> {
    @Override
    public void onApplicationEvent(ApplicationStartedEvent event) {
        
    }
}
```



Spring Container가 실행 되기 전 Event Listener

```JAVA
public class Listener implements ApplicationListener<ApplicationStartingEvent> {
    @Override
    public void onApplicationEvent(ApplicationStartingEvent event) {
        
    }
}
```

**이슈: Event가 Application Context 가 실행되기 전/후 중 언제 실행되는 지 유의할 것!**

Application Context가 실행 되기 전, StartedEvent :: 따로 Listener로 등록해주어야 한다.

```java
public static void main(String args) {
	SpringApplication app = new SpringApplication(SpringInitApplication.class);
	app.addListener(new Listener());
	app.run(args);
}
```

----

### Application Args

```
public static void main(String args) {
	// args : Application Argument
}
```

Application Argument

* Spring에서 Bean으로 등록해놓았다. 
* 가져다 쓰면 된다.

### ApplicaionRunner

```
@Component
public class Runner implements ApplicationRunner {
    @Override
    public void run(ApplicationArguments args) throws Exception {
		// Applicaion Argument Bean을 사용하는 예시
    }
}
```

Spring이 실행되면 위 run 메서드로 간다.

---

## 외부 설정

### 외부 설정 파일이란?

* 애플리케이션에서 사용하는 여러가지 설정 값들을 애플리케이션의 밖이나 안에 정의할 수 있는 기능을 말한다.

### application.properties 

* 이 파일은 스프링부트가 애플리케이션을 구동할 때 자동으로 로딩하는 파일이다.
* key - value 형식으로 값을 정의하면 애플리케이션에서 참조하여 사용할 수 있다.

```
tommy.name = tommy
tommy.age = ${random.int}
```

* 융퉁성 있는 바인딩
  * (케밥) tommy.full-name
  * (언드스코어) tommy.full_name
  * (캐멀) tommy.fullName
  * -> 알아서 tommy.fullName으로 바인딩 된다.
* 데이터 바인딩
  * properties에는 데이터 타입 정의되어 있지 않다.
  * 내가 바인딩할 곳에서 정의한 data type으로 바인딩 된다.

```
public class TestPropertyConfiguration {
	@NotBlank
    private String name; 
    // config 나 value : 사용할 곳에서 정의한 data type으로 바인딩된다.
    private int age;
```



cf.

```
heumsi.sessionTimeout = 20s
```

위와 같이 `s`, `m`, `d` 등과 같은 suffix 로 `Duration` 타입임을 명시하고 바인딩시켜줄 수 있다.



### 사용 방법

1. Value로 접근

```JAVA
 @Value("${tommy.name}")
 String name
```

* @Value로 접근 시 변수 값에 대한 Validation을 할 수 없다.



2. Property 통으로 접근
   * Validation 可
   * getter, setter 정의 해주어야 한다.

```JAVA
@Component
@ConfigurationProperties("tommy")
@Validated
public class TestPropertyConfiguration {
	@NotBlank
    private String name;
    private int age;

    public String getName() {
        return name;
    }
    
 	// ...	
}
```





Property는 어디에 정의하느냐, 어디에 저장하느냐에 따라 우선 순위가 달라진다.

**프로퍼티 우선 순위**

1. 유저 홈 디렉토리에 있는 spring-boot-dev-tools.properties
2. 테스트에 있는 @TestPropertySource
3. @SpringBootTest 애노테이션의 properties 애트리뷰트
4. 커맨드 라인 아규먼트
5. SPRING_APPLICATION_JSON (환경 변수 또는 시스템 프로티) 에 들어있는 프로퍼티
6. ServletConfig 파라미터
7. ServletContext 파라미터
8. java:comp/env JNDI 애트리뷰트
9. System.getProperties() 자바 시스템 프로퍼티
10. OS 환경 변수
11. RandomValuePropertySource
12. JAR 밖에 있는 특정 프로파일용 application properties
13. JAR 안에 있는 특정 프로파일용 application properties
14. JAR 밖에 있는 application properties
15. JAR 안에 있는 application properties
16. @PropertySource
17. 기본 프로퍼티 (SpringApplication.setDefaultProperties)

**application.properties 우선 순위 (높은게 낮은걸 덮어 씁니다.)**

1. file:./config/
2. file:./
3. classpath:/config/
4. classpath:/



cf. 테스트와 property

* 테스트 코드에 있는 property로 테스트가 진행된다.

  테스트 실행 = production code + resource (application.properties) 컴파일 => test code + resource (application.properties) 컴파일

* application-test.properties로 production code에서와 다른 이름의 property 파일을 저장하는 게 좋다.



cf. 3rd Party Configuration: Component 사용할 수 없다. (외부에서 정의한 )

@Bean은 가능, @ConfiguartionProperties -> 규칙

---

## Profile

### Profile이란 ? 

dev, prod, test : 스프링 부트에서는 프로파일(Profile)을 통해 스프링 부트 애플리케이션의 런타임 환경을 관리할 수 있습니다. 예로들어 어플리케이션 작동 시 테스트 환경에서 실행할 지 프로덕션 환경에서 실행할 지를 프로파일을 통해 관리할 수 있죠.



### 사용 방법

1. 어떤 property를 사용할지 명시한다. (at application.properties)
   ```
   [application.properties]
   spring.profiles.active=prod
   ```
2. 특정 Profile에 사용할 Bean을 명시한다.

   ```JAVA
   @Profile("prod")
   @Configuration
   public class BaseConfiguration {
   
       @Bean
       public String hello(){
           return "hello production";
       }
   }
   ```

* 결과

```JAVA
@Profile("test")
@Configuration
public class TestConfiguration {

    @Bean // 함수 Bean으로 등록 可
    public String hello(){ 
        return "hello test";
    }
}

@Component
public class AppRunner implements ApplicationRunner {

    @Autowired
    private String hello;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println("=============");
        System.out.println(hello); // prod로 Profile을 설정했으면 hello production이 출력된다.
        System.out.println("=============");
    }
}
```



### application-{프로파일}.properties 파일을 통한 프로파일 관리

**application-{프로파일}.properties** 파일을 생성하여 관리하게 되면 **application.properties** 보다 우선순위가 높게 외부 설정값이 관리되므로 보다 편하게 프로파일을 관리할 수 있습니다. 

```
[application.properties]
spring.profiles.active=prod
# application-prod.properties 가 우선순위가 높으므로 
# 아래 값은 다른 값으로 오버라이딩 된다.
saelob.name=power

[application-prod.properties]
saelobi.name=saelobi prod
# 추가할 프로파일 설정 -- properties에서 다른 profile 추가 가능하다.
spring.profiles.include=saelobi 

[application-saelobi.properties]
saelobi.fullName=KBS
```



<h2> 로깅 </h2>

```JAVA
@Component
public class AppRunner implements ApplicationRunner {
	// slf4j 로깅 파사드를 통해 logback 로깅 모듈을 지원
	private Logger logger = LoggerFactory.getLogger(AppRunner.class);

	@Override
	public void run(ApplicationArguments args) throws Exception {
		logger.info("=============");
		logger.info("This is Spring Boot App");
		logger.info("============="); // 기본적으로 출력된다.
        logger.debug("debug mode"); // 별도의 설정을 해야 출력된다.
        // [application.properties] logging.level.project.Runner=DEBUG
	}
}
```



### application.properties를 통한 로깅 설정

**application.properties를 통해 다음과 같이 로깅 설정을 할 수 있습니다.**

```
[application.properties]

spring.output.ansi.enabled=always
# 로그 메세지가 저장되는 로그 디렉터리 (새로 디렉토리가 생성되고, 여기에 log 정보가 저정되어 있다.)
logging.path=logs
# logging.level.{패키지 경로}를 통해 로깅 레벨을 결정할 수 있다.
# 원래 출력 안되던 logger.debug까지 출력된다.
logging.level.project.AppRunner=DEBUG
```
# 테스트

spring-boot-start-test

- test를 위한 기본
- gradle에 추가해주기

spring test에는 크게 2가지 종류가 있다.

1. 통합 테스트 (실제 서버 환경에서 테스트하기)
2. 슬라이싱 테스트 (실제 서버 환경 X에서 테스트하기)

## @SpringBootTest

- **실제 Spring 서버 구동 환경으로 하거나 (port 존재) / 그와 비슷한 환경 (Mock)에서 테스트한다**.
- @RunWith(SpringRunner.class) 와 같이 써준다.
- 실행하면 @SpringApplication 찾아간다
    - 모든 bean 스캔한다.
    - mock bean 설정하면 그 mock bean만 교체된다.
    - mock bean은 테스트할 때마다 reset된다.
### SpringBootTest  webEnvironment 1. MOCK (default)

- **내장 톰캣을 구동하지 않는다.**
    - servlet container를 띄우지 않는다.
    - spring 환경과 ""비슷""한 상태이다. (mockup 되어있다.)

    **→ MockMvc 같이 사용해준다.**

```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.MOCK)
@AutoConfigureMockMvc
public class Test {

    @Autowired
    MockMvc mockMvc;

    @org.junit.Test
    public void name() {
        mockMvc.perform(get("/hello"))
                .andExpect(status().isOk())
                .andExpect(content().string("hello keesun"))
                .andDo(print());

    }
}
```

### SpringBootTest  webEnvironment 2. Port 지정

- **실제로 내장 톰캣이 구동된다.**
    - Random_PORT or PORT 지정
        - Random_PORT로 테스트하는 게 더 좋다.
- TestRestTemplate : sync 동기
- **WebTestClinet** : async 비동기
    - API 테스트할 때 더 좋다. (성능상 이득)
    - gradle에 추가해야 한다.

```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@AutoConfigureMockMvc
public class Test {
    @Autowired
    TestRestTemplate restTemplate;

    @MockBean
    SampleService sampleService;
    
    @Test
    public void name() {
        String result = restTemplate.getForObject("/hello", String.class);
    }
}
```

## Slice Test

실제 Spring 환경이 아닌, 원하는 Component만 사용하고 싶을 때

### **@JsonTest**

- `@JsonTest`는 JSON의 직렬화, 역직렬화를 수행하는 라이브러인 Gson과 Jackson의 테스트를 제공합니다.

```
@RunWith(SpringRunner.class)
@JsonTest
public class BookJsonTest {

    @Autowired
    private JacksonTester<Book> json;

    @Test
    public void json_test() throws IOException {

        final Book book = new Book("title", 1000D);

        String content= "{\n" +
                "  \"id\": 0,\n" +
                "  \"title\": \"title\",\n" +
                "  \"price\": 1000\n" +
                "}";

        assertThat(json.parseObject(content).getTitle()).isEqualTo(book.getTitle());

        assertThat(json.write(book)).isEqualToJson("/test.json");
    }
}
```

### @WebMVCTest

**Client Test에 적합하다**.

web과 관련된 component만 등록된다.

### @OutputCapture

log 메세지에 찍히는 것으로 테스트할 수 있다.

---

**SpringDevTool 설정하면 Reload할 수 있다.**

### Restart vs Reload

써드 파티 안의 jar 파일은 base classloader로 읽어들이고, 사용자가 만드는 클래스는 restart classloader로 읽어들입니다. 이러한 접근은 재시작이 _cold start(완전히 껏다가 다시 켜는 것)_보다 훨씬 빨리 일어나게 도와줍니다.

아직 재시작(restart)가 느리게 느껴진다면, JRebel 같은 reloading 기술을 이용할 수도 있습니다. JRebel은 릴로딩이 편하도록 클래스를 다시 만듭니다.

---

# Spring Web MVC

WebMVC (Test) 자동설정이 있다. 

**SpringBoot MVC : 자동설정**을 제공한다.

**Spring MVC 프레임워크**는 디커플된 웹 애플리케이션 개발 방법을 제공한다. Dispatcher Servlet, ModelAndView, View Resolver 과 같은 단순개념을 이용해서 웹 애플리케이션 개발을 쉽게 할 수 있도록 해준다.

**[출처]** [Spring Boot vs. Spring MVC vs. Spring 의 비교](http://blog.naver.com/sthwin/221271008423)|**작성자** [멋진태혁](http://blog.naver.com/sthwin)

Spring MVC 확장: @Configuration + @WebMVC Configurer

Spring MVC 재정의: @Configuration + @EnableWebMvc

## HttpMessgaeConvertors

Http 요청 ( Request/Response Body ) ↔ Object

```java
@PostMapping("/user")
public @ResponseBody User create
```

## VeiwResolver

Request Accetp Header에 따라 결과가 달라진다.

1. 모든 view type에 따른 결과를 만든다. (by. HttpMessageConvertor)
    - json, xml 등 format
        - (만약 convertor에서 해당 type을 지원하지 않는다면 gradle에 추가해줄 것)
    - view: jsp, html 같은 타입?? 아니면 json, xml 같은 타입?? 헷갈림!
2. accept header에 맞는 결과를 돌려준다. (by. ViewResolver)

ViewResolver가 Convertor에 특정 type으로 만들라고 시킨다 ??

### 정적 리소스

[localhost:8080/me.html](http://localhost:8080/me.html) → static resoure file

서버에서 어떤 작업을 한 후 데이터를 제공하는 게 아니라 원래 있던 데이터를 제공하는 것

- 기본 리소스 위치가 있고 변경할 수도 있다.

- 브라우저: 자기가 받은 리소스 last modified 있음, 받으려는 파일의 last modified와 비교해서 변화가 있다면 다시 받고 아니라면 원래 있던 리소스를 재사용한다. (=304 응답)
- ResourceHttpRequestHandler가 처리한다.
    - Handler: 어떤 요청이 왔을 때 처리를 지정한다.
        - [localhost:8080/me.html](http://localhost:8080/me.html) → static resoure file **(기본)**
        - **/m/something에 대한 요청을 처리를 추가할 수 있다.**

    ```java
    @Configuration
    public class WebConfig implements WebMvcConfigurer {
        @Override
        public void addResourceHandlers(ResourceHandlerRegistry registry) {
            registry.addResourceHandler("/m/**")
                    .addResourceLocations("classpath:/m/")
                    .setCachePeriod(20);
        }
    }
    ```

### 웹 JAR

[https://velog.io/@jayjay28/웹-JAR](https://velog.io/@jayjay28/%EC%9B%B9-JAR)

**Client 측에서 사용하는 웹 JAR (ex. jQuery)를 Server Gradle에 추가해서 사용할 수 있다 !!**

```java
[Gradle]
implementation (group: 'org.webjars.bower', name: 'jquery', version: '3.3.1')

[html]
<script src="/webjars/jquery/3.3.1/dist/jquery.min.js"></script>
<script>
    $(function() {
        alert("ready!");
    })
</script>
```

템플릿 엔진: 주로 view를 만들 때 사용 (항상 X, 다른 것도 할 수 있다.)

- Spring에 의존성을 추가해주어야 한다.
- 동적 컨텐츠를 보여줄 때 의미가 있다.

JSP : 권장 X

- WAR 패키징 해야 된다.
- JAR 패키징 X
- Servlet 엔진 중에 WAR 지원하지 않는 것도 있어서 가급적 사용하지 않는 게 좋다.

## Thymeleaf

- 스프링 부트가 자동 설정을 지원하는 템플릿 엔진 中 1
- 템플릿 엔저니 ~ thymeleaf 를 사용하면 view rendering이 어떻게 되는지 알 수 있다.
- body에 <html> ... <.html> 어떻게 rendering되는지 나온다.
    - HTML Unit를 사용해서 rendering된 값을 테스트할 수 있다.

```java
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org"> // thymleaf를 사용한다.
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<h1 th:text="${name}">name</h1>
// * model에 name value가 넘어오면 그 value
// *  아니면 "name"이 나온다.
</body>
</html>
```

---

SpringBoot Test할 때 테스트 코드의 Package과 프로덕션 코드의 Package 경로를 고려해야 한다.

[java.lang.IllegalStateException: Unable to find a @SpringBootConfiguration, you need to use @ContextConfiguration or @SpringBootTest(classes=...) with your test](https://parkcheolu.tistory.com/125)

```java

/*
 test code의 package: java
 production class의 package: java/production
 --> 에러!
*/
@RunWith(SpringRunner.class)
@WebMvcTest(SampleController.class)
public class Test {
    @Test
    public void test() {
			// do test
    }
}
```
## HtmlUnit

- html 파일을 테스트하는 도구
    - html 파일의 element를 확인할 수 있다.
- webClient 만들어서 사용

xpaht: xml 테스트

- html도 xml 중 하나라서
- html의 elemetn를 확인할 수 있다.
- ex. h1의 값 확인

```java
@RunWith(SpringRunner.class)
@WebMvcTest(SampleController.class)
public class Test2 {
    @Autowired
    WebClient webClient;

    @org.junit.Test
    public void name() throws IOException {
        HtmlPage page = webClient.getPage("/hello");
        HtmlHeading1 h1 = page.getFirstByXPath("//h1");
        assertThat(h1.getTextContent()).isEqualTo("tommy");
    }
}
```

## ExceptionHandler

실제 Error를 관리하는 곳 : ErrorController

### Error Status Code에 따른 에러 페이지 만들기

resources에 error status code를 이름으로 갖는 페이지를 만든다.

ex. 404.html, 4xx.html

**ErrorViewResolver**: 좀 더 동적인 에러 페이즈를 만들 수 있다.

## HATEOAS

- REST 조건 3단계에 해당한다.
- 내가 요청한 **Rource와 연관있는 링크 정보 (url)을 알려준다.**
    - 내가 요청한 Resource에서 실행 가능한 링크 정보를 알려준다.
        - (예) 요청 결과: 잔고 30000원 → 입금, 출금 url
        - 요청 결과: 잔고 0원 → 입금 url
    - Client측에서는 그 링크로 요청할 수 있다.
    - 효과
        - Client가 Resource에 대한 상태를 알게 된다.
        - 즉  Client측에서 Server에 요청할 때, 될지 안될지 모르는 상태가 아니라 실행 가능한 상태라는 것을 확신한 상태에서 요청하게 된다.
        - Server측에서 Url을 바꿨을 때, Client측에서도 수정할 필요가 없다.
            - Client: Server에서 보내준 HATEOAS 실행 가능한 url로 접근하기 때문에!

- 의존성만 넣으면 사용할 수 있다.
- self (내가 요청한 페이지)
- **개발자가 관련 링크를 추가해준다**
    - 관련 링크를 보내주는 Resource를 만들고, 걔를 return 해준다.

```java
@GetMapping("/hello")
    public Resource<Hello>  hello() {
        Hello hello = new Hello();
        hello.setPrefix("Hey, ");
        hello.setName("Tommy");

        Resource<Hello> helloResource = new Resource<>(hello);
        helloResource.add(linkTo(methodOn(SampleController.class).hello()).withSelfRel());
        return helloResource;
    }

MockHttpServletResponse:
             Body = {"prefix":"Hey, ","name":"Tommy","_links":{"self":{"href":"http://localhost/hello"}}}
```

[RESTful, Stateless, HATEOAS 그리고 Passport](https://anster.tistory.com/163)

---

Object Mapper도 Custom해서 사용할 수 있다.

application.properties에 원하는 Object Mapper를 등록해준다.

---

## CORS

**Origin**: URI 스키마 — (http/https)  + hostname (localhos) + 포트 (8080)

SOP **Single Origin Policy : Origin끼리 교환이 안 된다는 원칙**

- 8080 port를 사용하는 Client 프로젝트에서 4040 port를 사용하는 프로젝트에  접근한다.

    → 안된다.

- 해결 방법: 브라우저에서 JSONP 사용 - 서버에서 CORS

**Cross-Origin Resource Sharing: Origin 끼리 교환이 된다.**

```java
@CrossOrigin(origins = "https://locathost:8080")
@GetMapping("/hello")
public String hello() {
	return hello;
}

// 8080 Client Application에서 4040 port를 사용하는 프로젝트에 접근할 수 있다. 
```

- 메서드에 CrossOrigin 설정을 할 수 있다.
- Controller에 CrossOrigin 설정을 할 수 있다.
- 프로젝트 자체에 CrossOrigin 설정을 할 수 있다.

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
	@Override
	public void addCorsMapping(CorsRegistry registry) {
		registry.addMapping("/hello")
						.allowedOrigins("https://localhost:8080");
	}
}
```

---

## Spring in-memory Database

(예) H2, HSQL

의존성 추가하면 자동적으로 인메모리 데이터베이스가 생긴다.

properties에 h2 console 허용해주고 console을 볼 수 있다.

## JDBC

Java DataBase Connectivity

- 종류에 상관없이 Database와 연결하기 위한 API
    - 실제 Database와 연결해주는 것은 JDBC Driver
1. Driver를 Load한다.
2. Database를 연결한다. Connection 
    - `Driver.getConnection()`  Driver가 Connection을 만든다.
3. SQL문을 실행한다.
4. Database 연결을 종료한다.

## Spring JDBC

→ JDBC를 사용할 때 Driver Load 및 Database Connection 입력없이 사용하는 방식

1. ~~Driver를 Load한다.~~
2. ~~Database를 연결한다. Connection. (Driver.getConnection())~~
3. SQL문을 실행한다. → 얘만 작성할 수 있게 하는 방법
4. ~~Database 연결을 종료한다.~~

(예) JDBC Template

## DBCP

DataBase Connection Pool

Database Connection 비용 크다 → connection pool : 미리 connection을 만들어 놓고, 가져다 쓴다.

- Connection by. WAS
- Database 관련 정보는
- 성능에 유의미하기 때문에 여러가지 설정을 잘 해야 한다.

## Data Source

- interface
- connection 생성 / 반납에 대해 정의하고 있다.

DataSource라는 객체는 **Connection Pool을 관리**하는 목적으로 사용되는 객체
Application에서 이 DataSource 객체를 통해 **Connection을 얻어오고 반납하는 작업**을 할 수 있다.

mysql connector 의존성을 추가하여, mySql을 사용할 수 있다. 

---

properties에 사용할 database 정보를 써놓는다.

spring.datasource.url = jdbc:mysq://localhost:3306/springboot

[spring.datasource.username](http://spring.datasource.name) = tommy

spring.datasource.password = password

→ 위 database를 사용할 것이고, username과 password는 다음과 같다.

---

# ORM

ORM : 객체와 릴레이션을 mapping할 때 개념적 불일치를 해결하는 프레임워크

개념적 불일치

1. **상속**
- 객체는 상속 可
- Table은 상속 不可

2. **식별자**

- 객체의 identity: hashcode 또는 equals
- Table의 identity: id

상속 관계 : Spring Data JPA → JPA → Hibernate → Datasource

### Spring Data JPA

- 의존성 추가 해주기

implementation 'org.springframework.boot:spring-boot-starter-data-jpa'

의존성 추가하고 따로 db추가하는 것 ??

`@DataJpaTest` : 테스트용으로 in-memory 데이터 베이스를 사용할 수 있다.

`@SpringBootTest`: Server Database가 바뀐다. → 인수테스트

**test code와 production code 다른 Database를 가지는 게 좋다.**

@SpringBootTest(properties overriding 가능)

```java
// 슬라이싱 테스트:  repository와 관련된 것만 테스트
@RunWith(SpringRunner.class)
@DataJpaTest // == 슬라이싱 테스트, in-memery가 반드시 필요하다.
public class AccountRepositoryTest {
    @Autowired
    DataSource dataSource;
```

```java
public interface AccountRepository extends JpaRepository<Account, Long> {
   Account findByUsername(String username);
   Optional<Account> findByUsername(String username); // Optional return도 可
}

* Account의 멤버변수로 Username 알아서 처리된다.
```

application.properties

[Spring에서 JPA / Hibernate 초기화 전략](https://pravusid.kr/java/2018/10/10/spring-database-initialization.html)

DDL(Data Definition Language)

- Create, Alter, Drop, Truncate

`spring.jpa.hibernate.ddl-auto`

→ Entity를 보고 스키마를 자동으로 만들어준다.

`spring.jpa.hibernate.ddl-auto=update` [옵션]

- 기존 데이터 유지, 바뀐 스카마 추가된다.
- 내가 String name → nickname 하면 name 있는 상태에서 nickname이 추가된다.

`spring.jpa.hibernate.ddl-auto=validate`

- entity를 보고 스키마를 만들어주지 않는다.
- 이미 스키마가 존재하는 상태에서 relation mapping이 되는지 검증해준다.

schema.sql → data.sql 순서로 실행된다.

- data platform에 맞는 schema, data.sql 사용할 수 있다.
- property에 platform 추가 해주어야 한다.

`spring.jpa.show-sql=true`

사용되는 sql 보인다.

## Data Imgration

### Flyway

data도 version 관리할 수 있는 툴

- Version 관리를 알아서 해주는 게 아니라 **내가 Version에 해당하는 sql을 만들어준다.** 해당 sql 을 실행한다.
    - table이나 초기 데이터 설정을 해준다.
- 의존성 추가
- resource/db/migration — V1__이름.sql , V2__이름.sql
    - 위 package에 version에 해당하는 sql을 직접 작성해준다.
    - **V1 실행, V2 ... 마지막 Version SQL 실행된다.** (마지막 Version만 실행 X)
    - 이미 실행했다면 해당 sql 절대로 건드리면 안된다. 새로 Version으로 sql을 만든다.
- table에 version 관리를 하는 flyway table이 생긴다.

# Spring Security

spring security 설정하면 **모든 request에 대해 인증을 요청**한다.

- 모든 요청을 가로채서 인증을 요구함
- (Spring Boot) Application 용 Uer를 생성하여, in-memory로 저장한다. — name, password
    - at UserDetailsService
    - 이 User 정보로 로그인할 수 있다.
    - 원래 Application 만들 때 이런 Uer 만들어주어야 한다.

**Accept Header에 따라 요구하는 인증의 형태가 달라진다.**

- 설정 x : Basic - 401 error
- text/html — form : Basic - 301 error
    - 3xx redirection으로 spring이 제공하는 로그인 페이지로 이동한다.

---

testImplementation 'org.springframework.security:spring-security-test'

@WithMockUser

→ Security 설정 된 상태에서 User Mock 처리 후 테스트 가능

---

로직 처리 없이 url - page rendering mapping

```java
		@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/hello").setViewName("hello");
    }
}
```

---

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                // if anyMatchers "/a". "/b" -- 해당 url에 match 되는 것은 permit
                .antMatchers("/", "/hello").permitAll()
                // else authenticated이 필요하고, login이 필요하다.
                .anyRequest().authenticated()
                .and()
                .formLogin()
                .and()
                .httpBasic();

    }
}
```

---

- Spring Security에서의 User username, password가 있는데
- **password 저장할 때는 Encoding 처리해주어야 한다.**
- Encoding된 것은 Server에서 처리이고, 실제 사용자는 원래의 PW로 사용한다.

---

### RestTemplate, WebClient

RestTemplateBuilder. WebClientBuilder를 Bean으로 등록해서 사용한다.

- RestTemplate
    - 동기
- WebClient
    - 비동기
- global customize하면 전역적으로 customizing된 WebClientBuilder를 사용할 수 있다.
- WebClient 자체를 전역 Bean으로 등록할 수 도 있다.

---

# Spring Actuator

spring boot actuator는 한 마디로 얘기하면 "**스프링 부트 애플리케이션**에서 제공하는 여러가지 정보(end point) 를 **모니터링**하기 쉽게 해주는 기능"이다.

다른 애플리케이션으로 접근할 때에 사용할 수 있다. (CORS)

- 여러가지 정보가 있는데 properties에서 공개 설정을 해준다.
    - api처럼 [localhost:8080/actuator/info](http://localhost:8080/actuator/info)  이런 식으로 접근하게 된다.
        - HTTP 접근할 거라면, 스프링 security로 보안해둘 것
    - 또는 jconsole 프로그램으로 접근할 수도 있다

@EnableAdminServer

- Client 애플리케이션에 대한 AdminServer를 설정할 수 있다.
    - AdminServer는 또 다른 애플리케이션이다.
- Client 애플리케이션 정보를 properties에 적어놓으면 Client 애플리케이션 정보를 편하게 볼 수 있다.
- AdminServer에도 Security 정할 것

## Factory

1. 객체의 생성 방법을 결정
2. 객체를 만들어서 돌려준다.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5575d369-7996-454c-8610-455ab5290488/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5575d369-7996-454c-8610-455ab5290488/Untitled.png)

- 애플리케이션의 컴포넌트 역할을 하는 오브젝트와 애플리케이션의 구조를 결정하는 오브젝트를 분리했다는 것이 의미가 있다.
- Factory를 통해 오브젝트를 전달하는 것 → 제어의 역전

## 제어의 역전

- 오브젝트가 자신이 사용할 오브젝트를 스스로 선택하지 않는다.
- 생성하지도 않는다.
- 자신이 사용할 오브젝트의 생성을 다른 대상에게 위임한다.

### 라이브러리와 프레임워크

- 라이브러리 사용: 제어권을 그 프로그램이 가지고 있다.
- 프레임워크 사용: 제어권이 프레임워크에게 있다.

### JAVA Bean

자바 빈은 특정 형태의 클래스를 가르키는 말이고 VO는 주로 계층형 구조에서 계층간 값을 전달하기 위해 자바 빈의 형태로 구현한 클래스입니다.

참고로, 원래는 자바 빈은 액티브 엑스나 DLL 처럼 컴포넌트화되어 서드 파티 개발툴이나 응용프로그램에서 가져다 쓸 수 있는 개념으로 출발했습니다.

하지만 데스크탑 자바가 지지 부진하면서 그런 개념이 널리 쓰이진 못하게 되고, 이후 전혀 관계없는 EJB가 마케팅적 이유로 이름을 이어 받게 되면서 본래의 의미는 거의 퇴색했습니다.

그래서 지금 시점에서는 그냥 **퍼블릭 생성자와 속성(컨벤션에 맞는 getter/setter 조합)을 갖는 클래스**를 가르키는 뜻으로 쓰이는 만큼, POJO(Plain Old Java Object)와도 거의 동일한 개념으로 이해해도 무방합니다.

## Bean Factory, Application Context, Spring Container

- 위 3개 모두 비슷한 개념
- Bean Factory & Application Context

- BeanFactory를 확장한 Application Context
    - ApplicationContext → (상속) → BeanFactory (interface)

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4f56915e-6aae-450c-96ae-21273ff91870/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4f56915e-6aae-450c-96ae-21273ff91870/Untitled.png)

- Client 는 Factory에 대한 정보를 알 필요가 없어진다.

### Application Context는 종합 IoC 서비스 제공한다.

- 단지 Object 생성 및 관계 설정만 ! (X)
- 오브젝트가 만들어지는 방식, 시점과 전략을 다르게 가져갈 수도 있고, 이에 부가적으로 자동생성, 오브젝트에 대한 후처리, 정보의 조합, 설정 방식의 다변화, 인터셉팅 등 오브젝트를 효과적으로 활용할 수 있는 다양한 기능을 제공한다. 또, 빈이 사용할 수 있는 기반기술 서비스나 외부 시스템과의 연동 등을 컨테이너 차원에서 제공해주기도 한다.

### 의존 관계

 A → B

- A가 B에 의존한다.
- B가 변하면 A에 영향을 미친다.
- (예) B의 메서드가 변하면 A도 변해야 한다.

### Spring  Bean의 생명주기

@PostConstruct : 생성 후 호출된다.

- 의존성이 제대로 주입되었는지 검사
- 실행시점의 custom 초기화 로직이 필요할 때
- 생성자에서 의존성 주입 검사를 하면 안된다.
    - 생성하는 시점에서는 Property 주입을 수행하지 않음

@PreDestory: 소멸 전 호출된다.

> ❗Scope가 Singleton일 때만 가능

## Component Scan

1. xml 설정 파일에 Bean 등록

    ```java
    <bean .. >
     <property name ="setter" ref="bean id">
    </bean>
    ```

2. Annotation & Component Scan

    > ❗컴포넌트 스캔은 하위 Package에서만 Scan된다.

## Autowired

1. 생성자에 붙이기
    ```java
    @Autowired
    public BookService(BookRepository bookRepository) {
        this.bookRepository = bookRepository;
    }
    ```

2. Autowired + setter ⇒ 의존성 추가한다는 의미

    ```java
    @Autowired
    public void setBookRepository(BookRepository bookRepository) {
        this.bookRepository = bookRepository;
    }
    ```

3. Field

    ```java
    @Autowired
    BookRepository bookRepository;
    ```

---

- @Autowired(requried=false)
    - 디폴트: requried = true
    - false로 설정하면, 의존성 주입 받지 않아도 정상적으로 생성된다.
- @Primary : 동일한 이름의 Bean 이 있을 때,우선적으로 등록해라.
    - @Autowired BookRepository
        - @Primary MyBookRepository implements BookRepository
        - HerBookRepository implements BookReposiroy
- 여러 개의 Bean 받을 수도 있다.

    @Autowired

    List<BookRepository> bookRepository;

    → Order 추가시 순서 지정 가능

## Bean Property

1. Singleton
2. Prototype
    - Request
    - Session
    - WebSocket

만약 싱글톤이 프로토타입을 참조하고 있다면, 싱글톤 객체를 생성할 때 프로토타입의 객체를 박아놓는다. 이 때 프로퍼티가 변경되지 않는다. 따라서 옛날 프로토 객체를 계속 보고 있는 것이다.

이걸 간단하게 쓰려면, @Scope(value=“prototype”, proxyMode = ScopedProxyMode.TARGET_CLASS) 을 씌우면 된다. 직접 참조하면 안 되고, 프록시를 거쳐야지만 프로토를 새롭게 할당해준다. 이 프록시 빈을 주입시켜주는 것이다.

이 방법이 싫다면, ObjectProvider proto; 와 같이 정의해주면 된다.

싱글톤은 항상 쓰레드 세이프가 아니다. A 쓰레드와 B 쓰레드 모두 같은 자원(객체)를 공유하기 때문에 싱글톤의 프로퍼티는 변경될 가능성이 농후하다. 따라서 쓰레드 세이프 한 방식을 고민해야 한다.

## Message Source

안녕하세요 /  Hello 이런 식으로 사용할 때

1. Property로 Message를 등록한다.
2. 등록된 Message 사용할 때

```java
@Autowired
MessageSource messageSource;

messageSoruce.getMessage("greeting")
```

## Application Event Publisher
Observer Pattern

1. Event Class를 만든다.

    ```java
    public class MyEvent {

        private int data;

        private Object source;

        public MyEvent(Object source, int data){
           this.source = source;
           this.data = data;
        }

        public Object getSource() {
            return source;
        }

        public void setSource(Object source) {
            this.source = source;
        }

        public int getData() {
            return data;
        }

        public void setData(int data) {
            this.data = data;
        }
    }
    ```

    - 이벤트는 POJO 스타일로 만들 수 있다. 원래는 ApplicationEvent를 상속받아 만들어야 했지만 Spring 4.2부터는 클래스를 상속받지 않아도 이벤트로 사용할 수 있다.
2. 이벤트 보내기

    ```java
    @Autowired
        ApplicationEventPublisher publisher;

        @Override
        public void run(ApplicationArguments args) throws Exception {
            publisher.publishEvent(new MyEvent(this, 10));
        }
    ```

3. 이벤트 받기

    ```java
    @Component
    public class MyEventHandler {
    	@EventListener
    	public void handle(MyEvent event){
    		System.out.println("Event received " + event.getData());
    	}
    }
    ```

    @EventListener + Parameter의 Event 보고 알아서 이벤트 받아온다.

- 복잡한 프로그램에서 사용하면 의존성을 줄일 수 있다.
- 스프링에서 제공하는 기본 이벤트에서도 사용

    ●  ContextRefreshedEvent: ApplicationContext를 초기화 했더나 리프래시 했을 때 발생.

    ●  ContextStartedEvent: ApplicationContext를 start()하여 라이프사이클 빈들이 시작

    신호를 받은 시점에 발생.

    ●  ContextStoppedEvent: ApplicationContext를 stop()하여 라이프사이클 빈들이 정지

    신호를 받은 시점에 발생.

    ●  ContextClosedEvent: ApplicationContext를 close()하여 싱글톤 빈 소멸되는 시점에

    발생.

    ●  RequestHandledEvent: HTTP 요청을 처리했을 때 발생.

## ResourceLoader

리소스를 읽어오는 기능을 제공하는 인터페이스

ApplicationContext extends ResourceLoader

리소스 읽어오기

- 파일 시스템에서 읽어오기
- 클래스패스에서 읽어오기
- URL로 읽어오기
- 상대/절대 경로로 읽어오기