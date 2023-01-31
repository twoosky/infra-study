# AWS ECR: Elastic Container Registry
## ECR 이란
* AWS 공식 문서에서는 ECR을 아래와 같이 정의하고 있다.
* Private Docker Hub와 유사
> Amazon Elastic Container Registry (Amazon ECR)는 안전하고 확장 가능하고 신뢰할 수 있는 AWS 관리형 컨테이너 이미지 레지스트리 서비스입니다.
> Amazon ECR는 AWS IAM를 사용하여 리소스 기반 권한으로 프라이빗 컨테이너 이미지 리포지토리를 지원합니다.   
> 이렇게 하면 지정된 사용자 또는 Amazon EC2 인스턴스가 컨테이너 리포지토리 및 이미지에 액세스할 수 있습니다.  
> 선호하는 CLI를 사용하여 도커 이미지, Open Container Initiative(OCI) 이미지 및 OCI 호환 아티팩트를 푸시, 풀 및 관리할 수 있습니다.

## 실습
* ECR Repository에 Docker 이미지를 올려보자

### 1. ECR IAM 권한 추가
* ECR 서비스를 이용하려면 IAM 사용자에게 ECR 접근 권한을 주어야 한다.
* IAM 사용자에게 `AmazonEC2ContainerREgistryFullAccess` 권한 부여
<img src="https://user-images.githubusercontent.com/50009240/215730112-5544acc4-b477-4737-8ab8-ad3280a94f44.png">

### 2. ECR Repository 생성
* AWS ECR 서비스에 들어가 Repository 생성
* 아래와 같이 설정 후 나머지는 default로 놓고 생성
<img src="https://user-images.githubusercontent.com/50009240/215730748-704a4c0c-beb0-4de6-af16-00847ec40ae1.png">

* 생성한 Repository에 들어가 푸시 명령보기를 클릭하면 아래와 같이 명령어들이 나온다.
* 이를 통해 Dcoker Image를 ECR에 push할 수 있다.
<img src="https://user-images.githubusercontent.com/50009240/215731839-5ed70cb9-7238-4cb1-af2a-1f0a5c7869a7.png">

## 3. EC2에 AWS CLI, Docker 설치
1. AWS CLI 설치
2. aws configure 환경변수 설정 (access key, secret key)
3. docker 설치
4. 푸시 명령 순서대로 입력


