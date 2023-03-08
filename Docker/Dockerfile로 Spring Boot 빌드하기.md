# Dockerfile로 Spring Boot 빌드하기

### 1. Sample 프로젝트 만들기
```java
@RestController
public class HelloController {

    @GetMapping("/")
    public String home() {
        return "hello world";
    }
}
```
### 2. Dockerfile 만들기


### 3. Dockerfile로 이미지 빌드하기
```
$ docker build -t docker-example:0.0.1 .
```

### 4. 이미지 실행하기
```
$ docker run docker-example:0.0.1
```
