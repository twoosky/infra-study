AWS 정리 자료
===
인프런 ['스스로 구축하는 AWS 클라우드 인프라 - 기본편'](https://www.inflearn.com/course/aws-%ED%81%B4%EB%9D%BC%EC%9A%B0%EB%93%9C-%EC%9D%B8%ED%94%84%EB%9D%BC-%EA%B8%B0%EB%B3%B8/dashboard) 강의 정리 자료입니다. 

Amazon S3와 CloudFront로 정적 파일 배포
===
* Amazon S3
  * 대표적인 storage 서비스
  * S3 Bucket 안에 html, 그림, 동영상 파일 등의 컨텐츠를 업로드하여 웹 호스팅 설정을 하면 웹사이트처럼 작동하도록 구성 가능
  * S3만으로도 웹 호스팅이 가능 - 객체 스토리지 서비스의 특징
  * 웹 브라우저에서 읽어들여야할 컨텐츠의 크기가 커지면 로딩이 길어짐.  
* Amazon CloudFront
  * 대표적인 컨텐츠 전송 네트워크 서비스
  * 컨텐츠를 빠르게 읽을 수 있도록 `caching` 기능 제공
  * 전세계 AWS 각 리전의 엣지 로케이션에 리소스의 복사본을 미리 로드해놓고, 사용자들이 짧은 지연 시간에 파일을 받을 수 있도록 함
  * S3보다 가속화된 웹서비스 개발 가능  
* 아키텍처 구현 순서
  1. s3 정적 웹 호스팅 구성하기
     * S3 Bucket 생성
     * 외부에서도 접근 가능하도록 정적 웹 사이트 호스팅 활성화
     * index.html 컨텐츠 업로드
     * 웹 사이트 엔드포인트 테스트  
  2. CloudFront를 이용해 웹 사이트 속도 높이기
     * CloudFront 배포 만들기
     * 생성된 CloudFront 도메인 확인
* S3 웹 호스팅
  * s3 Bucket 생성
  * 정적 웹사이트 호스팅
    * `Properties(속성)`에서 `정적 웹사이트 호스팅` 활성화
    * 정적 웹사이트 호스팅의 Index document에 `index.html` 입력
      * Bucket에 웹호스팅 되어 접속했을 때 가장 처음으로 읽어들이는 파일 명시
  * 권한 설정
    * `Block public access`: S3 접속에 대한 보안 설정
    * 'Bucket Policy`: 외부 인터넷으로부터 aws 내부의 접근을 차단/허용, Bucket, Bucket 내 컨텐츠에 대한 보안 설정
    * 생성한 Bucket Policy를 복사해 Bucket Policy editor에 붙여넣음  
    ```json
     {
    "Version": "2012-10-17",
    "Id": "Policy1633880232920",
    "Statement": [
        {
            "Sid": "Stmt1633880224183",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:*",
            "Resource": "arn:aws:s3:::haneul-web/*"
         }
       ]
     } 
     ```
     * `Statement` 내 `Resource` 끝에 `/*`를 무조건 붙여줘야 됨.
       * Bucket 안에 있는 모든 컨텐츠는 외부로 나가는 것을 허용한다는 의미
       * `/*`가 없으면 Bucket에 컨텐츠를 올려도 웹 호스팅이 안됨.
  * Overview(객체)에 컨텐츠 업로드
  * EndPoint 테스트
     * Overview에서 각 컨텐츠 클릭해 객체 URL로 접속
* CloudFront 웹 가속화
  * CloudFront 배포 생성
     * 원본 도메인으로 Bucket 선택
     * 설정의 `Price Class`에서 CloudFront의 Edge Location 사용 범위를 설정할 때, 캐싱할 지역 지정
  * 생성된 CloudFront의 도메인 이름으로 캐싱되고 있는 S3의 컨텐츠를 받아오는지 확인
     * 예시) index3.html의 이미지를 CloudFront를 통해 엑세스 테스트
       * index3.html에서 CloudFront 배포 도메인 이름으로 이미지에 엑세스 할 URL 입력
           * `<img src="https://{CloudFront 도메인}/awslogo.png" alt="AWS LOGO"/>`
       * Bucket에 index3.html 업로드
       * `{CloudFront 도메인}/index3.html`로 확인
       * * 캐싱된 awslogo.png를 엣지 로케이션으로부터 가져와 이미지를 띄움.
       * `크롬 개발자 도구 -> 네트워크`에서 index3.html 선택하고 `Response Headers`의 `X-Caches`값이 Hit from cloudfront이면 cloudfront으로부터 즉, 엣지 로케이션에서 컨텐츠를 불러 왔다는 의미
  * 참조: https://musma.github.io/2019/06/29/publish-static-assets-over-https-using-cloudfront.html
  * 참조: https://aws.amazon.com/ko/blogs/korea/amazon-s3-amazon-cloudfront-a-match-made-in-the-cloud/
* 질문
  * 그러니까 CloudFront를 사용하면 버킷의 컨텐츠들을 각 리전의 엣지 로케이션에 저장하고, CloudFront의 도메인 이름을 통해서 엣지 로케이션에 저장되어 있는 컨텐츠들을 꺼내올 수 있다는 건가?
  * 엣지 로케이션이 캐시 역할인건가?
  * 그럼 그냥 버킷의 컨텐츠에 접근하는 것보다 CloudFront를 통해서 엣지 로케이션에 있는 컨텐츠에 접근하는 것이 더 빠르다는건가?
     
     
     
     
     
     
     
