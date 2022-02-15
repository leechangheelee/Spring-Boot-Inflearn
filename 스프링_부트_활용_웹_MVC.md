## **스프링 부트 활용**
  * 스프링 웹 MVC - 소개
    * 스프링 웹 MVC
      * https://docs.spring.io/spring-framework/docs/5.0.7.RELEASE/spring-framework-reference/web.html#spring-web
    * 스프링 부트 MVC
      * 자동 설정으로 제공하는 여러 기본 기능 (앞으로 살펴 볼 예정)
      ```java
      /* user/UserController.java (src/main 아래) */
      ...
      @RestController
      public class UserController {

          @GetMapping("/hello")
          public String hello() { // 메서드 이름은 아무렇게나 지어도 됨
              return "hello";
          }

          /*
          스프링 웹 MVC 기능을 우리는
          아무런 설정파일을 작성하지 않아도
          스프링 웹 MVC 개발을 바로 시작할 수 있다.
          이것이 스프링 부트가 제공해주는 기본 설정 때문임.
          spring-boot-autoconfigure 라는 모듈에 spring.factories 에 보면 WebMvcAutoConfiguration 라는 클래스가 있음.
          요게 적용되면서 가능한 이야기.
           */
      }
      ```
      ```java
      /* user/UserControllerTest.java (src/test 아래) */
      ...
      @RunWith(SpringRunner.class)
      @WebMvcTest(UserController.class)
      public class UserControllerTest {

          @Autowired
          MockMvc mockMvc;
          // 이 객체는 @WebMvcTest 애노테이션을 사용하면 자동으로 빈으로 만들어주기 때문에 우리가 그냥 빈에 있는걸 바로 꺼내쓸 수 있음

          @Test
          public void hello() throws Exception {
              mockMvc.perform(get("/hello"))
                      .andExpect(status().isOk())
                      .andExpect(content().string("hello"));
          }
      }
      ```
    * 스프링 MVC 확장
      * @Configuration + WebMvcConfigurer
      ```java
      /* config/webConfig.java */
      ...
      @Configuration
      // @EnableWebMvc ← 이러면 스프링 부트가 제공하는 MVC 기능은 다 사라지고 개발자가 MVC 설정을 아래에서 직접 다 해줘야함 (재정의)
      public class WebConfig implements WebMvcConfigurer {
          /*
           스프링 부트가 제공해주는 MVC 기능들을 다 사용하면서
           거기에 추가적으로 설정을 더 하고싶을 때
           */
      }
      ```
    * 스프링 MVC 재정의
      * @Configuration + @EnableWebMvc
***
  * 스프링 웹 MVC - HttpMessageConverters
    * 스프링 프레임워크에서 제공하는 인터페이스. 스프링 MVC의 일부분. HttpMessageConverter 는 여러가지가 있고, 어떤 요청을 받았는지 / 어떤 응답을 보내야 하는지에 따라 사용하는 HttpMessageConverter 가 달라짐. (예. JSON 요청, JSON 본문 일 때 JSON 메세지 컨버터가 사용됨)
    * 참고) https://docs.spring.io/spring-framework/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-config-message-converters
    * HTTP 요청 본문을 객체로 변경하거나, 객체를 HTTP 응답 본문으로 변경할 때 사용.  
      `{"username":"changhee", "password":"123"}` ↔ User
      * @RequestBody
      * @ReponseBody
      ```java
      /* user/User.java */
      ...
      public class User {

          private Long id;

          private String username;

          private String password;


          // 자바 빈 규약에 따라서 getter, setter 를 사용해서 데이터 바인딩을 해준다
          public Long getId() {
              return id;
          }

          public void setId(Long id) {
              this.id = id;
          }

          public String getUsername() {
              return username;
          }

          public void setUsername(String username) {
              this.username = username;
          }

          public String getPassword() {
              return password;
          }

          public void setPassword(String password) {
              this.password = password;
          }
      }
      ```
      ```java
      /* UserController.java */
      ...
      @RestController
      public class UserController {

          @GetMapping("/hello")
          public String hello() {
              // public @ResponseBody String hello() 와 동일. @RestController 에노테이션을 붙이면 @ResponseBody 생략해도 됨
              // @RestController 이 아니라 @Controller 애노테이션을 붙이면 뷰 네임 리졸버를 사용해서 아래 이름 ("hello") 의 뷰를 찾으려고 시도하게 됨
              // @RestController 붙이면 "hello" 가 메세지 컨버터를 타서 응답본문으로 내용이 들어가게 됨
              return "hello";
          }

          @PostMapping("/users/create")
          public User create(@RequestBody User user) {
          // public @ResponseBody User create(@RequestBody User user) 와 동일
          // @RestController 에노테이션을 붙이면 @ResponseBody 생략해도 됨
              // Composition 타입 (클래스 안에 여러가지 프로퍼티를 가짐) 인 경우 기본적으로 JSON 메세지 컨버터가 사용됨
              // 그냥 리턴타입이 String 이다 → String 메세지 컨버터가 사용됨. int 도 마찬가지.
              return user;
          }
      }
      ```
      ```java
      /* UserControllerTest.java */
      ...
      @RunWith(SpringRunner.class)
      @WebMvcTest(UserController.class)
      public class UserControllerTest {

          @Autowired
          MockMvc mockMvc;
          // 이 객체는 @WebMvcTest 애노테이션을 사용하면 자동으로 빈으로 만들어주기 때문에 우리가 그냥 빈에 있는걸 바로 꺼내쓸 수 있음

          @Test
          public void hello() throws Exception {
              mockMvc.perform(get("/hello"))
                      .andExpect(status().isOk())
                      .andExpect(content().string("hello"));
          }

          @Test
          public void crateUser_JSON() throws Exception {
              String userJSON = "{\"username\":\"changhee\", \"password\":\"123\"}";
              mockMvc.perform(post("/users/create")
                          .contentType(MediaType.APPLICATION_JSON_UTF8)
                          .accept(MediaType.APPLICATION_JSON_UTF8) // 응답으로 어떤 데이터를 원하느냐
                          .content(userJSON)) // 요청 본문
                      .andExpect(status().isOk())
                      .andExpect(jsonPath("$.username",
                              is(equalTo("changhee"))))
                      .andExpect(jsonPath("$.password",
                              is(equalTo("123"))))
                      .andDo(print());
          }
      }
      ```
***
  * 스프링 웹 MVC - ViewResolver
    * ContentNegotiatingViewResolver
      * 뷰 리졸버중의 하나인데, 들어오는 요청의 accept 헤더에 따라 응답이 달라짐
      * accept header (테스트 코드의 .accept(MediaType.APPLICATION_XML) 부분) ← 브라우저 또는 클라이언트가 어떠한 타입의 본문을 응답을 원한다 라고 서버한테 알려주는 것
      * 참고) https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-multiple-representations
      * 경우에 따라서는 accept header 를 제공하지 않는 요청도 있는데, 그럴때는 format 이라는 매개변수 사용.  
        `/path?format=pdf`
    * 스프링 부트
      * 뷰 리졸버 설정 제공
      * HttpMessageConvertersAutoConfiguration
        ```java
        /* UserControllerTest.java */
        ...
        /* 요청을 JSON 으로 보내고 응답을 XML로 받는 테스트 */
        @Test
        public void crateUser_XML() throws Exception {
            String userJSON = "{\"username\":\"changhee\", \"password\":\"123\"}";
            mockMvc.perform(post("/users/create")
                            .contentType(MediaType.APPLICATION_JSON_UTF8)
                            .accept(MediaType.APPLICATION_XML) // 응답으로 어떤 데이터를 원하느냐
                            .content(userJSON)) // 요청 본문
                    .andExpect(status().isOk())
                    .andExpect(xpath("/User/username")
                            .string("changhee"))
                    .andExpect(xpath("/User/password")
                            .string("123"))
                    .andDo(print());
        }
        ...
        ```  
        `HttpMediaTypeNotAcceptableException` 발생 : 미디어 타입을 처리할 http message converter 가 없는 경우.  
        HttpMessageConvertersAutoConfiguration → JacksonHttpMessageConvertersConfiguration 확인.
        @ConditionalOnClass({XmlMapper.class}) 조건 (클래스패스에 XmlMapper.class 가 있을때만 등록이 되도록 설정) 에 따라 MappingJackson2XmlHttpMessageConverterConfiguration 컨버터가 등록이 안됨.  
        아래와 같이 의존성을 추가하여 클래스패스에 xml 매퍼 등록.
        ```xml
        <!-- pom.xml -->
        <dependency>
            <groupId>com.fasterxml.jackson.dataformat</groupId>
            <artifactId>jackson-dataformat-xml</artifactId>
            <version>2.9.6</version>
        </dependency>
        ```
        * 요즘은 주로 JSON 을 사용하기 때문에 보통은 별도의 설정없이 사용
***
  * 스프링 웹 MVC - 정적 리소스 지원
    * 정적 리소스 : 웹브라우저나 클라이언트에서 요청이 들어왔을 때 해당하는 리소스가 이미 만들어져 있고, 만들어져 있는 리소스를 그대로 보내주면 되는 경우
    * 정적 리소스 맵핑 "/**" (URL 패턴이 루트부터 매핑)
      * 기본 리소스 위치
        * classpath:/static
          * 예) "/hello.html" → (src/main/resource)/static/hello.html
        * classpath:/public
        * classpath:/resources
        * classpath:/META-INF/resources
        * spring.mvc.static-path-pattern: 맵핑 설정 변경 가능  
          ```
          #application.properties
          spring.mvc.static-path-pattern=/static/**
          # URL 패턴이 /static 부터 매핑
          # http://localhost:8080/static/hello.html 로 접근해야 함
          # 딱히 이렇게 쓰는 경우는 잘 없음
          ```
      * Last-Modified 헤더를 보고 304 응답을 보냄 (브라우저 개발자도구로 확인)
        * If-Modified-Since 이후에 내용이 바뀌었으면 새로 달라 (200 응답)
        * 그 이후 내용 변경없이 refresh 시 리소스를 새로 보내지는 않음. 응답이 빠름 (캐싱 동작. 304 응답)
      * ResourceHttpRequestHandler가 처리함
        * WebMvcConfigure의 addResourceHandlers로 커스터마이징 할 수 있음
          ```java
          /* config/WebConfig.java */
          ...
          @Configuration
          public class WebConfig implements WebMvcConfigurer {

              @Override
              public void addResourceHandlers(ResourceHandlerRegistry registry) {
                  // resource handler 를 추가하는 것
                  // 스프링 부트가 제공해주는 리소스 핸들러는 그대로 유지하면서
                  // 개발자가 원하는 리소스 핸들러만 따로 추가할 수 있음
                  registry.addResourceHandler("/m/**")
                          .addResourceLocations("classpath:/m/") // 반드시 "/" 로 끝나야 함
                          .setCachePeriod(20); // 초 단위. 캐시 컨트롤을 직접 해주어야 함
              }
          }
          ```
          * "/m/hello.html" → (src/main/resource)/m/hello.html
***
  * 스프링 웹 MVC - 웹JAR
    * 웹JAR 맵핑 "/webjars/**"
      * 웹JAR : 클라이언트에서 사용하는 자바스크립트 라이브러리들 (jQuery, Bootstrap, Vue.js, React.js, Angular 등등) 을 JAR 파일로 추가할 수 있음. JAR파일로 의존성 추가하고 템플릿을 사용해서 동적으로 콘텐츠를 생성할 때 또는 정적 리소스에서도 웹JAR에 있는 css나 JavaScript를 참조할 수 있음.
      * 의존성 추가 관련, maven 중앙저장소에서도 검색가능
        ```xml
        <!-- pom.xml -->
        <!-- https://mvnrepository.com/artifact/org.webjars.bower/jquery -->
        <dependency>
            <groupId>org.webjars.bower</groupId>
            <artifactId>jquery</artifactId>
            <version>3.3.1</version>
        </dependency>
        ```
        ```html
        <!-- static/hello.html -->
        <!DOCTYPE html>
        <html lang="en">
        <head>
            <meta charset="UTF-8">
            <title>Title</title>
        </head>
        <body>
        Hello Static Resource HAHAHA LOL

        <!--<script src="/webjars/jquery/3.3.1/dist/jquery.min.js"></script>-->
        <!--버전을 빼려면 의존성에 webjars-locator-core 를 추가해야 함-->
        <script src="/webjars/jquery/dist/jquery.min.js"></script>
        <script>
            $(function() {
                alert("ready!");
            });
        </script>

        </body>
        </html>
        ```
      * 버전 생략하고 사용하려면
        * webjars-locator-core 의존성 추가
          ```xml
          <!-- pom.xml -->
          <!-- https://mvnrepository.com/artifact/org.webjars/webjars-locator-core -->
          <dependency>
              <groupId>org.webjars</groupId>
              <artifactId>webjars-locator-core</artifactId>
              <version>0.35</version>
          </dependency>
          ```
***
  * 스프링 웹 MVC - index 페이지와 파비콘
    * 웰컴 페이지
      * index.html 찾아 보고 있으면 제공
      * index.템플릿 찾아 보고 있으면 제공
      * 둘 다 없으면 에러페이지
    * 파비콘
      * favicon.ico (정적 리소스 경로에 위치시키면 됨)
      * 파비콘 만들기 https://favicon.io
      * 파비콘이 안바뀔 때?
        * https://stackoverflow.com/questions/2208933/how-do-i-force-a-favicon-refresh
***
  * 스프링 웹 MVC - Thymeleaf
    * 스프링 부트가 자동 설정을 지원하는 템플릿 엔진
      * FreeMarker
      * Groovy
      * __Thymeleaf__
      * Mustache
    * JSP를 권장하지 않는 이유
      * JAR 패키징 할 때는 동작하지 않고, WAR 패키징 해야 함
      * Undertow는 JSP를 지원하지 않음
      * https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#web.servlet.embedded-container.jsp-limitations
    * Thymeleaf 사용하기
      * https://www.thymeleaf.org/
      * https://www.thymeleaf.org/doc/articles/standarddialect5minutes.html
      * 의존성 추가 : spring-boot-starter-thymeleaf
        ```xml
        <!-- pom.xml -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
        ```
      * 템플릿 파일 위치 : /src/main/resources/__templates__/
      * 예제 : https://github.com/thymeleaf/thymeleafexamples-stsm/blob/3.0-master/src/main/webapp/WEB-INF/templates/seedstartermng.html
      ```html
      <!-- templates/hello.html -->
      <!DOCTYPE html>
      <html lang="en" xmlns:th="http://www.thymeleaf.org">
      <head>
          <meta charset="UTF-8">
          <title>Title</title>
      </head>
      <body>
      <!--기본적으로 name 값이 전달이 안되면 Name 이 출력됨-->
      <h1 th:text="${name}">Name</h1>
      </body>
      </html>
      ```
      ```java
      /* SampleController.java */
      ...
      @Controller
      public class SampleController {

          @GetMapping("/hello")
          public String hello(Model model) { // 여기서는 리턴하는 String 이 뷰 이름이 되는 것. @RestController 사용때가 응답본문으로 들어가는 것.
              // 뷰에다가 전달해야 하는 데이터들은 model 이라는 곳에 담을 수 있음. MAP 이라고 생각하면 됨
              model.addAttribute("name", "changhee");
              return "hello";
          }

      }
      ```
      ```java
      /* SampleControllerTest.java */
      ...
      @RunWith(SpringRunner.class)
      @WebMvcTest(SampleController.class)
      public class SampleControllerTest {

          @Autowired
          MockMvc mockMvc;

          @Test
          public void hello() throws Exception {
              // 요청 "/hello"
              // 응답
              // - 모델 name : changhee
              // - 뷰 이름 : hello

              mockMvc.perform(get("/hello"))
                      .andExpect(status().isOk())
                      .andExpect(view().name("hello"))
                      .andExpect(model().attribute("name", is("changhee")))
                      .andExpect(content().string(containsString("changhee")))
                      .andDo(print());
              // 응답본문에서 뷰 렌더링 결과(html) 확인이 가능한 것은 Thymeleaf 를 쓰니까 가능한 것. (JSP와 달리 서블릿 엔진 개입없이 뷰 결과 확인가능)
              // Thymeleaf 는 독자적으로 최종적인 뷰를 완성함
          }

      }
      ```
***
  * 스프링 웹 MVC - HtmlUnit ← html을 단위테스트 하기 위한 툴
    * HTML 템플릿 뷰 테스트를 보다 전문적으로 하자
      * https://htmlunit.sourceforge.io/
      * https://htmlunit.sourceforge.io/gettingStarted.html
      * 의존성 추가
        * test scope → 테스트할 때에만 사용됨
        ```xml
        <!-- pom.xml -->
        <dependency>
            <groupId>org.seleniumhq.selenium</groupId>
            <artifactId>htmlunit-driver</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>net.sourceforge.htmlunit</groupId>
            <artifactId>htmlunit</artifactId>
            <scope>test</scope>
        </dependency>
        ```
      * @Autowire WebClient
        ```java
        /* SampleControllerTest.java */
        ...
        @RunWith(SpringRunner.class)
        @WebMvcTest(SampleController.class)
        public class SampleControllerTest {

            @Autowired
            WebClient webClient;

            @Test
            public void hello() throws Exception {
                HtmlPage page = webClient.getPage("/hello");
                HtmlHeading1 h1 = page.getFirstByXPath("//h1");
                assertThat(h1.getTextContent()).isEqualToIgnoringCase("changhee");
            }

        }
        ```
        * MockMvc 도 같이 사용가능
***
  * 스프링 웹 MVC - ExceptionHandler
    * 스프링 애노테이션 기반 MVC 예외처리 방법
      * @ControllerAdvice ← 여러 컨트롤러에서 발생하는 예외를 한곳에서 처리 가능
      * @ExceptionHandler
        ```java
        /* SampleException.java */
        ...
        public class SampleException extends RuntimeException {

        }
        ```
        ```java
        /* AppError.java */
        ...
        public class AppError {

            private String message;

            private String reason;

            public String getMessage() {
                return message;
            }

            public void setMessage(String message) {
                this.message = message;
            }

            public String getReason() {
                return reason;
            }

            public void setReason(String reason) {
                this.reason = reason;
            }
        }
        ```
        ```java
        /* SampleController.java */
        ...
        @Controller
        public class SampleController {

            @GetMapping("/hello")
            public String hello() {
                throw new SampleException();
            }

            @ExceptionHandler(SampleException.class)
            // 이 SampleContoller 안에서 발생하는 SampleException 이라는 예외가 발생하면 이 핸들러를 사용하겠다.
            public @ResponseBody AppError sampleError(SampleException e) { // @ResponseBody 사용해서 JSON 으로 리턴
                AppError appError = new AppError(); // 이 앱에 특화되어있는, 에러에 대한 정보를 담고 있는 커스텀한 클래스 (필수는 아님)
                appError.setMessage("error.app.key"); // 보통은 받아온 e 활용해서 처리하면 됨
                appError.setReason("IDK IDK IDK");
                return appError;
            }
        }
        ```
    * 스프링 부트가 제공하는 기본 예외 처리기
      * BasicErrorController
        * HTML과 JSON 응답 지원
        * 커스텀한 에러 처리가 없으면 맨 마지막에 BasicErrorController 가 처리
      * 커스터마이징 방법
        * ErrorController 구현
    * 커스텀 에러 페이지
      * 상태 코드 값에 따라 에러 페이지 보여주기
      * src/main/resources/static|templates/error/
      * html 파일 명이 상태값과 동일해야 함
        * 404.html ← 404 에러 발생시
        * 5xx.html ← 500번대 에러 발생시
      * ErrorViewResolver 구현 ← 좀 더 동적으로 에러페이지 처리가능
***
  * 스프링 웹 MVC - Spring HATEOAS
    * Hypermedia As The Engine Of Application State
      * 서버 : 현재 리소스와 연관된 링크 정보를 클라이언트에게 제공한다.
      * 클라이언트 : 연관된 링크 정보를 바탕으로 리소스에 접근한다.
      * 연관된 링크 정보
        * Relation
        * Hypertext Reference
        * 예) https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FrzVz6%2FbtqY5MDjtur%2FYgmaTlZjevjwZN9nkpM2m1%2Fimg.png
      * spring-boot-starter-hateoas 의존성 추가
        ```xml
        <!-- pom.xml -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-hateoas</artifactId>
        </dependency>
        ```
      * https://joomn11.tistory.com/26
      * https://spring.io/guides/gs/rest-hateoas/
      * https://docs.spring.io/spring-hateoas/docs/current/reference/html/#reference
    * ObjectMapper 제공 (@Autowired 로 ObjectMapper 주입받아 사용 가능)
      * ↑ 여러가지 이유로 많이 사용하게 됨. 우리가 제공하는 리소스를 JSON 으로 변환할 때 사용하는 인터페이스. 잭슨 이라는 라이브러리에서 제공해 주는 클래스이고 의존성에 spring-boot-starter-web 만 있어도 빈으로 등록해줌
      * spring.jackson.*  
        application.properties 에서 ObjectMapper를 커스터마이징 해서 사용가능
      * Jackson2ObjectMapperBuilder
    * LinkDiscovers 제공
      * 클라이언트 쪽에서 링크 정보를 Rel 이름으로 찾을 때 사용할 수 있는 XPath 확장 클래스
***
  * 스프링 웹 MVC - CORS
