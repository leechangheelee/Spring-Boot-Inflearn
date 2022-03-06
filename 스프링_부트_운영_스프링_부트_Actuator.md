## **스프링 부트 운영**
  * 스프링 부트는 애플리케이션 운영 환경에서 유용한 기능을 제공함
  * 스프링 부트가 제공하는 엔드포인트와 메트릭스 그리고 그 데이터를 활용하는 모니터링 기능에 대해 학습
***
  * 스프링 부트 Actuator - 소개
    * https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#production-ready-endpoints
    * 의존성 추가
      ```xml
      <!-- pom.xml -->
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-actuator</artifactId>
      </dependency>
      ```
    * 애플리케이션의 각종 정보를 확인할 수 있는 Endpoints
      * 다양한 Endpoints 제공
      * JMX 또는 HTTP를 통해 접근 가능함
      * shutdown을 제외한 모든 Endpoint는 기본적으로 __활성화__ 상태
      * 활성화 옵션 조정
        * `management.endpoints.enabled-by-default=false`
        * `management.endpoint.info.enabled=true`
***
  * 스프링 부트 Actuator - JMX와 HTTP
    * 참고
***
  * 스프링 부트 Actuator - 스프링 부트 어드민
    * 참고
