# Docker Compose 사용 이유
먼저 Docker Compose를 사용하지 않고, Spring boot, Redis 컨테이너를 만들어 컨테이너간 통신을 해보자.  
이를 위해, 페이지를 리프레쉬했을때 숫자 0부터 1씩 계속 증가하는 간단한 프로젝트를 구현 해보자.

## 1. Spring Boot 환경 세팅
1. build.gradle에 아래 의존성을 추가하고 빌드해준다.
```
implementation 'org.springframework.boot:spring-boot-starter-data-redis'
```
2. application.yml 파일에 Redis host와 port를 설정한다.
```yml
spring:
  data:
    redis:
      host: localhost
      port: 6379
```
3. RedisTemplate 사용을 위한 기본 Configuration 작성
```java
@Configuration
public class RedisConfig {

    @Value("${spring.redis.host}")
    private String host;

    @Value("${spring.redis.port}")
    private int port;

    @Bean
    public RedisConnectionFactory redisConnectionFactory() {
        return new LettuceConnectionFactory(host, port);
    }

    @Bean
    public RedisTemplate<?, ?> redisTemplate() {
        RedisTemplate<?, ?> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(redisConnectionFactory());
        return redisTemplate;
    }
}
```
4. 어플리케이션 코드 작성
```java
@RestController
@RequiredArgsConstructor
public class HelloController {
    private final RedisTemplate<String, Integer> redisTemplate;

    @PostConstruct
    public void init() {
        redisTemplate.opsForValue().set("number", 0);
    }
    
    @GetMapping("/")
    public String hello() {
        Integer num = redisTemplate.opsForValue().get("number");
        redisTemplate.opsForValue().set("number", num + 1);
        
        return "숫자가 1씩 올라갑니다. 숫자: " + num;
    }
}
```
* `@PostConstruct`: 해당 객체 내 모든 의존성(Bean)들이 초기화 된 직후 딱 한 번만 실행되도록 해주는 어노테이션
* 해당 API 요청이 들어올 때마다 Redis를 이용하여 숫자 1씩 증가

## 2. Dockerfile 작성
Spring Boot 프로젝트를 실행시키기 위한 Dockerfile이다.
```dockerfile
FROM openjdk:17-alpine

WORKDIR /usr/src/app

COPY ./build/libs/demo-0.0.1-SNAPSHOT.jar ./build/libs/demo-0.0.1-SNAPSHOT.jar

CMD ["java", "-jar", "./build/libs/demo-0.0.1-SNAPSHOT.jar"]
```

## 3. Spring Boot, Redis 컨테이너 실행
1. Spring Boot 프로젝트 build
```
$ ./gradlew build
```
2. Redis 컨테이너 실행
```
$ docker run redis
```
3. Dockerfile 빌드 및 Docker Image 실행
```
$ docker build . -t springbootapp
```
```
$ docker run -p 5000:8080 springbootapp
```
그 후 localhost:5000 으로 요청을 보내면 아래와 같이 **Redis 연결에 실패**했다는 오류가 발생한다.
<img src="https://user-images.githubusercontent.com/50009240/224291462-8318c367-c239-417d-9bd7-bee48f70e25c.png" width="800" height="150">




## 4. Docker compose를 사용해 에러 해결
* 에러 발생 이유  
  * Docker compose를 사용하지 않고, 컨테이너 통신시 에러가 발생한다. 그 이유는 컨테이너간 통신할 때 네트워크를 연결해주지 않았기 때문이다.  
  * 컨테이너간 통신을 할 때 아무런 설정을 해주지 않으면, 서로 접근이 불가능하다.
* 어떻게 하면 컨테이너 간 통신을 할 수 있을까?  
  * CLI 명령어에서 --link와 같은 것들을 사용할 수도 있겠지만, 
  * 이러한 멀티 컨테이너 상황에서 쉽게 네트워크를 연결시켜주기 위해서는 Docker Compose를 이용하는 것이 좋다.

1. Docker Compose 파일 작성하기
```yml
version: "3"
services:
  redis-server:
    image: "redis"
  springboot-app:
    build: .
    ports:
      - "5000:8080"
```
