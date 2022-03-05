## **스프링 부트 활용**
  * 스프링 REST 클라이언트 - RestTemplate과 WebClient
    * RestTemplate
      * Blocking I/O 기반의 Synchronous API
      * ResteTemplateAutoConfiguration
      * 프로젝트에 spring-web 모듈이 있다면 RestTemplateBuilder 를 빈으로 등록해줌
        ```xml
        <!-- pom.xml -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        ```
      * https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#rest-client-access
    * WebClient
      * Non-Blocking I/O 기반의 Asynchronous API
      * WebClientAutoConfiguration
      * 프로젝트에 spring-webflux 모듈이 있다면 Webclient.Builder 를 빈으로 등록해줌
        ```xml
        <!-- pom.xml -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-webflux</artifactId>
        </dependency>
        ```
      * https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-client
***
  * 스프링 REST 클라이언트 - 커스터마이징
    * 참고
