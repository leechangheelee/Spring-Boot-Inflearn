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
      ```java
      /* RestRunner.java */
      ...
      // (SampleController) REST API의 클라이언트 쪽이라고 가정
      @Component
      public class RestRunner implements ApplicationRunner {

          @Autowired
          RestTemplateBuilder restTemplateBuilder;

          @Override
          public void run(ApplicationArguments args) throws Exception {
              RestTemplate restTemplate = restTemplateBuilder.build();

              StopWatch stopWatch = new StopWatch();
              stopWatch.start();

              // Blocking I/O 기반의 Synchronous API
              String helloResult = restTemplate.getForObject("http://localhost:8080/hello", String.class); // 이 호출이 끝나기 전까지는 다음 라인으로 넘어가지 않음
              System.out.println(helloResult);

              String worldResult = restTemplate.getForObject("http://localhost:8080/world", String.class);
              System.out.println(worldResult);

              stopWatch.stop();
              System.out.println(stopWatch.prettyPrint());
          }
      }
      ```
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
      ```java
      /* RestRunner.java */
      ...
      // (SampleController) REST API의 클라이언트 쪽이라고 가정
      @Component
      public class RestRunner implements ApplicationRunner {

          @Autowired
          WebClient.Builder builder;

          @Override
          public void run(ApplicationArguments args) throws Exception {
              WebClient webClient = builder.build();

              StopWatch stopWatch = new StopWatch();
              stopWatch.start();

              // Non-Blocking I/O 기반의 Asynchronous API
              Mono<String> helloMono = webClient.get().uri("http://localhost:8080/hello")
                      .retrieve() // 결과 값을 가져와라
                      .bodyToMono(String.class); // 모노타입으로 변경을 해라
              // 요 작업을 해줘야 고여있는 스트림의 데이터가 흐름
              // subscribe를 해야 그제서야 실제 요청 보내고 가져오기 함. subscribe도 non blocking 이라 실행하고 바로 넘어감
              helloMono.subscribe(s -> {
                  // Asynchronous 한 콜백 형태로 응답이 오면 이 블럭 안의 코드가 실행됨
                  System.out.println(s);

                  if (stopWatch.isRunning()) {
                      stopWatch.stop();
                  }

                  System.out.println(stopWatch.prettyPrint());
                  stopWatch.start();
              });

              Mono<String> worldMono = webClient.get().uri("http://localhost:8080/world")
                      .retrieve()
                      .bodyToMono(String.class);

              worldMono.subscribe(s -> {
                  System.out.println(s);

                  if (stopWatch.isRunning()) {
                      stopWatch.stop();
                  }

                  System.out.println(stopWatch.prettyPrint());
                  stopWatch.start();
              });
          }
      }
      ```
    ```java
    /* SampleController.java */
    ...
    @RestController
    public class SampleController {

        @GetMapping("/hello")
        public String hello() throws InterruptedException {
            Thread.sleep(5000l);
            return "hello";
        }

        @GetMapping("/world")
        public String world() throws InterruptedException {
            Thread.sleep(3000l);
            return "world";
        }
    }
    ```
***
  * 스프링 REST 클라이언트 - 커스터마이징
    * 참고
