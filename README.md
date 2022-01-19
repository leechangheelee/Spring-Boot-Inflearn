# 스프링 부트 개념과 활용
  * 인프런 강의 수강 (업무활용)
  * https://www.inflearn.com/course/스프링부트

## **스프링 부트 시작하기**
  * 스프링 부트 소개
    * https://docs.spring.io/spring-boot/docs/2.0.3.RELEASE/reference/htmlsingle/#getting-started-introducing-spring-boot
  * 스프링 부트 시작하기
    * Maven 빌드로 시작
    * 참고) https://docs.spring.io/spring-boot/docs/2.6.2/maven-plugin/reference/htmlsingle/#?
  * 스프링 부트 프로젝트 생성기
    * https://start.spring.io/ 에서 프로젝트 generate 후 인텔리제이에서 import
  * 스프링 부트 프로젝트 구조
    * 메이븐 기본 프로젝트 구조와 동일
      * 소스 코드 (src/main/java)
      * 소스 리소스 (src/main/resource)
      * 테스트 코드 (src/test/java)
      * 테스트 리소스 (src/test/resource)
    * 메인 애플리케이션 위치
      * 기본 패키지
      * 참고) https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#using.structuring-your-code

## **스프링 부트 원리**
  * 의존성 관리 이해
    * 참고) https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#using-boot-dependency-management
  * 의존성 관리 응용
    * 버전 관리 해주는 의존성 추가
      * spring 에서 관리되는 의존성은 버전명을 명시해주지 않아도 따라옴. (인텔리제이에 아이콘 표시됨)
    * 버전 관리 안해주는 의존성 추가
      * 예) https://mvnrepository.com/ 의 modelmapper
      * 버전명을 명시해 주어야함. 서버에 운영중인 의존성도 버전을 명시해서 맞춰주는게 좋음.
    * 기존 의존성 버전 변경하기
      * 프로젝트의 pom.xml 에서 해당 속성 오버라이딩 해주면 됨
      * 예)
        ```xml
        <properties>
            <spring.version>5.0.6.RELEASE</spring.version>
        </properties>
        ```
  * 자동 설정 이해
    * @EnableAutoConfiguration (@SpringBootApplication 안에 숨어있음)
    * 빈은 사실 두 단계로 나눠서 읽힘
      * 1단계: @ComponentScan
      * 2단계: @EnableAutoConfiguration
    * @ComponentScan : 디폴트 패키지 부터 하위 패키지까지 자바파일 읽어서 아래 애노테이션이 붙은것들을 자바 빈으로 등록해줌 (다른 패키지는 등록 X)
      * @Component
      * @Configuration @Repository @Service @Controller @RestController
    * @EnableAutoConfiguration
      * spring.factories
        * org.springframework.boot.autoconfigure.EnableAutoConfiguration
          * 기본적으로 이 아래에 정의된 것들은 @Configuration 을 모두 달고 있다.
          * 그 중 조건에 따라 빈에 등록이 될수도 있고 안될 수도 있다.
        * @Configuration
          * Auto Configuration 도 Configuration 이다.
          * @Configuration : 스프링에서 빈을 등록하는 자바 설정파일
          * 참고) 애노테이션 위치는 위아래 상관없다.
        * @ConditionalOn~ : 조건에 따라 아래에 기술된 자바를 빈으로 등록하기도 하고 안등록 하기도 하고
          * @ConditionalOnMissingBean : 특정 빈 등록 안되어있으면 아래에 기술된 자바를 빈으로 등록 해준다.
  * 자동 설정 만들기
    * Starter와 AutoConfigure
      * xxx-Spring-Boot-Autoconfigure 모듈 : 자동 설정
      * xxx-Spring-Boot-Starter 모듈 : 필요한 의존성 정의
      * 그냥 하나로 만들고 싶을 때
        * xxx-Spring-Boot-Starter
          * 예) lee-spring-boot-starter
      * 구현 방법
        1. 의존성 추가
           ```xml
           <!-- pom.xml -->
           <dependencies>
               <dependency>
                   <groupId>org.springframework.boot</groupId>
                   <artifactId>spring-boot-autoconfigure</artifactId>
               </dependency>
               <dependency>
                   <groupId>org.springframework.boot</groupId>
                   <artifactId>spring-boot-autoconfigure-processor</artifactId>
                   <optional>true</optional>
               </dependency>
           </dependencies>

           <dependencyManagement>
               <dependencies>
                   <dependency>
                       <groupId>org.springframework.boot</groupId>
                       <artifactId>spring-boot-dependencies</artifactId>
                       <version>2.0.3.RELEASE</version>
                       <type>pom</type>
                       <scope>import</scope>
                   </dependency>
               </dependencies>
           </dependencyManagement>
           ```
        2. @Configuration 파일 작성
           ```java
           /* HollomanConfiguration.java */
           @Configuration
           public class HolomanConfiguration {

               @Bean
               public Holoman holoman() {
                   Holoman holoman = new Holoman();
                   holoman.setHowLong(5);
                   holoman.setName("Changhee");
                   return holoman;
               }

           }
           ```
        3. src/main/resource/META-INF에 spring.factories 파일 만들기
           ```
           org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
           me.changhee.HolomanConfiguration
           ```
        4. mvn install
           * 인텔리제이 > maven > Lifecycle > install 로 jar 생성
             * 이 프로젝트를 빌드해서 jar 파일 생성된걸 다른 maven 프로젝트에서도 갖다 쓸 수 있도록 로컬 maven 저장소에 설치함
             * 다른 프로젝트의 pom.xml 에서 의존성 설정하면 가져다 빈 등록해서 씀
               ```xml
               <!-- 다른 프로젝트의 pom.xml -->
               <dependency>
                   <groupId>me.changhee</groupId>
                   <artifactId>lee-spring-boot-starter</artifactId>
                   <version>1.0-SNAPSHOT</version>
               </dependency>
               ```
               * 다른 프로젝트에서 빈을 별도로 다시 정의하면 @ComponentScan 후 @EnableAutoConfiguration 통해서 빈 등록하면서 가져온걸로 덮어짐
