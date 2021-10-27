VPC와 중계서버(Bastion)
===

<img src= "https://user-images.githubusercontent.com/50009240/139061987-44386d07-e67b-48da-b71a-cad7c997ba9c.png" width="700" height="400">

* 아키텍처 다이어그램
  * subnet은 외부 인터넷과 연결 가능한 public subnet과 외부 인터넷과 연결되지 않는 private subnet이 있다.
  * EC2는 외부 인터넷에서 접속이 불가능한 private subnet에 위치하고, Internet Gateway와 ELB-ALB를 통해 EC2로 트래픽이 들어온다.
  * EC2가 외부 api와 통신하는 경우 즉, 트래픽이 외부로 나가야 하는 경우 라우팅 테이블에 NAT Gateway를 업데이트하여 외부와 통신한다.
  * EC2에 접근하기 위해선 중계서버 `Bastion`을 사용힌다.
    * 외부에서 접근 가능한 public subnet에 서버를 생성하여 user 또는 개발자가 해당 서버를 통해서 private subnet에 있는 EC2에 접근할 수 있도록 하는 중계서버를 `Bastion`이라고 한다.
