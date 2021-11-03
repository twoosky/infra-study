# RDS for MySQL 생성하기
<img src="https://user-images.githubusercontent.com/50009240/140083612-877ec9ab-0abd-4555-a6b5-faff1a72d083.png" width="700" height="400">

* 아키텍처 다이어그램
  * `RDS`: 애플리케이션 내에서 관계형 데이터베이스의 설정, 운영, 스케일링을 단순케 하도록 설계된 클라우드 내에서 동작하는 웹 서비스이다.
  * 서로 다른 가용영역인 private subnet에 RDS가 배치되어 있다.
  * 위쪽의 RDS는 `master RDS`이고, 아래 쪽의 RDS는 장애나 백업을 대비하기 위한 `대기 RDS`이다. 기본적으로 마스터 RDS로 접속하여 시스템이 운영 된다.
  * 만일 master RDS에 장애가 발생한다면 ***대기 RDS가 자동으로 master RDS가 되어***  웹 어플리케이션 시스템을 유지할 수 있게 한다.
  * 외부 인터넷을 통해 들어오는 request(데이터/query)는 `인터넷 게이트 웨이 -> 로드 밸런서 -> EC2`를 거쳐서 RDS에 도착한다.
  * request가 들어왔을 때 쿼리문에 대한 결과값을 반대 경로를 통해서 일반 유저나 개발자로 전송한다.
