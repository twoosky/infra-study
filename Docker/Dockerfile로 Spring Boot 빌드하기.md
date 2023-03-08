# Dockerfile로 Spring Boot 빌드하기
기본적으로 Spring Boot 프로젝트를 빌드하는 명령어는 다음과 같다.
```bash
$ ./gradlew build
$ java -jar build/libs/{프로젝트 이름}-0.0.1-SNAPSHOT.jar
```
**1. build를 통해 jar 파일 생성하기**  

<img src="https://user-images.githubusercontent.com/50009240/223781002-c2a235f9-2db4-4f94-b51a-ace7e81d7574.png" width="700" height="80">

그 결과 build/libs/ 경로에 demo-0.0.1-SNAPSHOT.jar 파일이 생성된다. (demo: 프로젝트 이름)

<img src="https://user-images.githubusercontent.com/50009240/223782062-6f0048bd-0185-4bae-ba1a-1e5a301023cd.png" width="300" height="170">

**2. jar 파일 실행하기**  

<img src="https://user-images.githubusercontent.com/50009240/223781467-8aa6f6d0-6bbb-4225-843e-ab1ed2e3423d.png" width="800" height="280">

Dockerfile을 통해 Docker Container 내부에서 Spring Boot Application jar를 실행해보자

## 1. Sample 프로젝트 만들기

* Gradle
* 3.0.4
* Java 17
```java
@RestController
public class HelloController {

    @GetMapping("/")
    public String home() {
        return "hello world";
    }
}
```
http://localhost:8080/ 으로 요청을 보내면 아래와 같이 작동한다.

<img src="https://user-images.githubusercontent.com/50009240/223785587-72dfdfc4-9549-4621-86fb-eee1da2b7dd3.png" width="300" height="80">

## 2. Dockerfile 만들기
```dockerfile
FROM openjdk:17-alpine

WORKDIR /usr/src/app

COPY ./build/libs/demo-0.0.1-SNAPSHOT.jar ./build/libs/demo-0.0.1-SNAPSHOT.jar

CMD ["java", "-jar", "./build/libs/demo-0.0.1-SNAPSHOT.jar"]
```
* `FROM`: 베이스 이미지
* `WORKDIR`: 컨테이너에서 작업할 디렉터리 지정 
* `COPY`: 빌드한 jar 파일을 컨테이너 내부에 복사
* `CMD`: 컨테이너에서 실행할 명령어

<details>
<summary>WORKDIR 사용 이유</summary>
<div markdown="1">

* WORKDIR 을 지정하지 않으면, 소스 파일들이 '/' 경로에 들어가게 된다.  
* 이때, Base Image가 가진 파일과 COPY 하는 파일의 이름이 동일하다면 Base Image의 폴더가 덮어씌워진다.  
* 따라서 애플리케이션을 위한 모든 소스 파일들은 working directory에 따로 보관하는 것이 좋다.  

</div>
</details>

<details>
<summary>COPY 사용 이유</summary>
<div markdown="1">

* Docker 이미지는 **파일 스냅샷**과 명령어를 가지고 있다. Docker 이미지를 통해 Dcoekr 컨테이너를 실행할 때,     
* 컨테이너 내부의 하드디스크에는 Docker 이미지의 **파일 스냅샷**만이 존재한다.   
* 이외 로컬에서 생성한 폴더, 파일들은 컨테이너 내부가 아닌 외부에 존재한다.      
* 따라서 컨테이너 내부에는 빌드한 jar 파일이 없다. 따라서 COPY를 통해 컨테이너 내부에 복사해줘야 한다.  

</div>
</details>


## 3. Dockerfile로 이미지 빌드하기
```
$ docker build . -t springbootapp
```
* `build .`: 현재 디렉터리의 Dockerfile 실행
* `-t`: 이미지에 이름 부여  

아래와 같이 Docker 이미지가 생성된다.  

<img src="https://user-images.githubusercontent.com/50009240/223791223-419b8f5a-ffa5-4614-9d8f-8409c6d21e5d.png" width="600" height="70">


## 4. 이미지 실행하기
* `-p` 옵션을 통해 로컬 네트워크와 컨테이너 내부 네트워크를 연결시켜줘야 한다.
```bash
$ docker run -p 50000:8080 springbootapp

# $ docker run -p 로컬 포트번호:컨테이너 내부 포트번호 이미지 이름
```
그 결과, http://localhost:50000/ 으로 요청을 보내면 정상 작동한다.

<img src="https://user-images.githubusercontent.com/50009240/223795217-a0bcbeee-1a19-4d49-9237-44e025e382a3.png" width="300" height="80">
