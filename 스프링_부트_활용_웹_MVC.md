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
    * 스프링 부트
      * 뷰 리졸버 설정 제공
      * HttpMessageConvertersAutoConfiguration
***
  * 스프링 웹 MVC - 정적 리소스 지원
***
  * 스프링 웹 MVC - 웹JAR
***
  * 스프링 웹 MVC - index 페이지와 파비콘
***
  * 스프링 웹 MVC - Thymeleaf
***
  * 스프링 웹 MVC - HtmlUnit
***
  * 스프링 웹 MVC - ExceptionHandler
***
  * 스프링 웹 MVC - Spring HATEOAS
***
  * 스프링 웹 MVC - CORS
