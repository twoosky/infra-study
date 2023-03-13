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

WORKDIR /build

COPY ./build/libs/demo-0.0.1-SNAPSHOT.jar demo-0.0.1-SNAPSHOT.jar

CMD ["java", "-jar", "demo-0.0.1-SNAPSHOT.jar"]
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
<img src="https://user-images.githubusercontent.com/50009240/224291462-8318c367-c239-417d-9bd7-bee48f70e25c.png" width="900" height="180">


## 3. 에러 발생 이유
* 컨테이너간 통신할 때 네트워크를 연결해주지 않았기 때문이다.  
* 컨테이너간 통신을 할 때 아무런 설정을 해주지 않으면, 서로 접근이 불가능하다.
* 레디스 클라이언트: Spring boot 내 `RedisTemplate`
<img src="https://user-images.githubusercontent.com/50009240/224770365-a90cee07-560b-4e57-8244-858149615899.png" width="570" height="190">

**어떻게 하면 컨테이너 간 통신을 할 수 있을까?**
* docker-compose를 사용하면 컨테이너간 통신을 할 수 있다.
* 기본적으로 Docker Compose는 하나의 default 네트워크에 모든 컨테이너를 연결한다.
* default 네트워크 안에서 컨테이너간 통신은 service 이름이 호스트명으로 사용된다.
## 4. Docker compose를 사용해 에러 해결
**1. Docker Compose 파일 작성하기**
```yml
version: '3'
services:
  redis:
    image: redis:alpine
    ports:
      - "6379:6379"
  springboot:
    build: .
    environment:
      - SPRING_DATA_REDIS_HOST=redis
    ports:
      - "8080:8080"
```
* `version`: 도커 컴포즈의 버전
* `services`: 실행하려는 컨테이너들 정의
* `image`: 사용할 이미지 이름
* `build`: 해당 경로에 있는 Dockerfile을 사용하여 빌드
* `environment`: 환경변수 설정 (application.yml의 spring.data.redis.host 환경변수 설정)
* `ports`: 로컬포트와 컨테이너포트 매핑
> docker-compose 내 컨테이너는 default 네투워크로 연결되어 있어 **service명 으로 컨테이너 간 통신**을 할 수 있다.  
> 따라서 springboot 컨테이너와 redis 컨테이너가 통신하기 위해선 spring.data.redis.host의 값을 redis로 설정해야 한다.

**2. Docker compose 실행하기**
```
$ docker-compose up -d
```
그 결과 localhost:8080 으로 요청 시 아래와 같이 나온다. 프로젝트 구현 성공!  

<img src="https://user-images.githubusercontent.com/50009240/224768368-4c52e4f2-1648-4d03-b861-05cbd04804f2.png" width="370" height="100">
