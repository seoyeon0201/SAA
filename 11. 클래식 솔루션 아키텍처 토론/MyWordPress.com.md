## Stateful Web App:MyWordPress.com

#### 서비스 설명

- scalable한 WordPress website
- 웹사이트에 접근하고 업로드한 사진이 올바르게 나타나길 원함
- 사용자 데이터와 블로그 내용은 MySQL DB에 저장되어야함

#### RDS Layer

> scalable한 WordPress website를 만들고 싶음

- RDS가 있는 계층 생성

#### Scaling with Aurora: Multi AZ & Read Replicas

> scalable한 WordPress website를 만들고 싶음

- `Aurora MySQL로 교체`
  - Multi AZ, Read Replicas 가능
  - Aurora를 사용함으로써 연산을 줄일 수 있음
  - 확장성 있고 기록하기 쉬움

#### Storing images with EBS

> EBS는 단일 EC2 instance 이미지 저장 시 정상 동작

- 하나의 EC2 인스턴스가 있고 하나의 EBS 볼륨이 연결되어 있는 단일 AZ의 단순한 솔루션 아키텍처

  - Load balancer에 연결되어 있음
  - 사용자는 load balancer로 이미지를 보내고 싶음
  - 이미지는 EBS Volume에 도달해 저장
  - 이미지를 불러올 때도 EBS volume에서 이미지를 읽어 사용자에게 전송

> EBS를 가지고 있는 아키텍처에서 확장하면 이미지 저장에 문제 발생

- 2개의 EC2 인스턴스와 2개의 AZ => 각각의 EC2 인스턴스는 각각의 EBS volume을 가짐
  - 사용자가 load balancer에 이미지를 전송하면 2개의 instance 중 랜덤으로 이미지가 보내지고 해당 인스턴스와 연결된 EBS volume에 이미지 저장
  - EBS volume은 하나의 인스턴스만 있을 때는 잘 작동하지만, 다중 AZ 또는 다중 인스턴스로 확장하면 오류 발생

#### Storing images with EFS

> EBS가 다중 AZ 또는 다중 인스턴스로 확장되면 문제 발생: 이미지를 불러올 때 이미지가 저장된 volume이 아닌 다른 volume에서 읽어온다면 오류 발생

- EFS = NFS
- `EFS`는 탄력적인 네트워크 인터페이스를 위해 각각의 AZ에 ENI 생성하고, 이 ENI는 EFS 드라이브에 접근하는 모든 EC2에 사용 가능
  - EFS Storage가 모든 인스턴스에 공유
  - 사용자가 M5 인스턴스로 이미지를 보내면, 이미지는 ENI를 거쳐 EFS로 전달되고 이미지가 EFS에 저장
  - 이미지를 불러오고 싶다면 인스턴스에서 ENI, ENI에서 EFS에 도달해 읽어옴
  - 가용 영역이나 인스턴스 수에 관계없이 모든 인스턴스가 동일한 파일에 접근하는 것을 허용해 다수의 EC2 인스턴스에 걸쳐 웹사이트 storage를 확장하는 흔한 방법


#### Service 적용 개념
- Aurora Database to have easy Multi-AZ and Read-Replicas
- Storing data in EBS (single instance application)
- Storing data in EFS (distributed application)
    - EBS가 EFS보다 저렴