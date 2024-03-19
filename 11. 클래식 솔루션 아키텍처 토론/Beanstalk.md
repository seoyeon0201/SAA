## Typical Archiecture:Web App 3-Tier

- loadbalancer가 있고 이곳에서 사용자의 모든 요청을 받음
- 다수의 Availability zone과 Auto Scaling group 존재
- 각각의 AZ(Availability zone)에 EC2 인스턴스 배포
- 백엔드에는 data subnet 존재
  - RDS DB(데이터 읽고 쓰기 가능), ElastiCache(캐싱 레이어 필요)

## Developer problems on AWS

> 우리가 배포할 애플리케이션이 많고 그들이 동일한 아키텍처를 따른다면 매번 생성하기 힘듬

- 개발자로서 인프라를 관리하고 코드를 배포하기 어려움
- 데이터베이스와 로드밸런서 설정이 번거로움
- 모든 것이 scaling 되기를 원함

- 대부분의 웹 애플리케이션은 ALB(로드밸런스)와 ASG(오토스케일링그룹)이 있는 동일한 아키텍처를 가짐

- 모든 개발자는 그냥 코드를 실행하기를 원함
- 다른 프로그래밍 언어로 개발하고 다양한 애플리케이션 환경을 가진다면 한 가지 방법으로 애플리케이션을 배포하길 원함

## Elastic Beanstalk

- 위의 개발자가 가지는 모든 문제 해결 가능

#### Elastic Beanstalk 개요

- 개발자 입장에서 애플리케이션을 AWS에 배포
- 하나의 인터페이스에서 EC2, ASG, ELB, RDS 등 모든 컴포넌트를 재사용하는 개념
- Beanstalk가 사용자를 대신해 모든 것을 배포하는 관리형 서비스 역할을 함
  - 용량 프로비저닝, 로드밸런서 설정, 스케일링, 애플리케이션 상태 모니터링,인스턴스 설정 등 처리
  - 개발자는 코드 자체만 담당하면 됨
- 모든 컴포넌트 설정에 대한 통제가 가능한데, 이것이 Beanstalk에서 하나의 인터페이스 번들로서 제공
- 간단하게 application 업데이트하는 방법 제공
- Beanstalk 서비스 자체는 무료지만, Beanstalk가 활용하는 인스턴스나 ASG, ELB에 대해서는 비용 지불

#### Elastic Beanstalk Component

- Application
  - 환경, 버전, 설정 등 Beanstalk 컴포넌트 집합
- Application Version

  - 사용자가 작성한 application code 버전과 동일
  - 버전1, 버전2, 버전3

- Environment
  - 특정한 애플리케이션 버전을 실행하는 리소스 집합
  - Tier: web server environment tier + worker environment tier
  - dev,test,prod 등 다양한 환경 생성 가능

> (1) Application 생성 > (2) 버전 Upload > (3) 환경 실행 > (4) 환경 관리. 반복하고 싶으면 새로운 버전을 upload(2)하는 단계로 가 버전 업데이트하면 우리 환경에서 새 버전 배포해 application stack 업데이트

#### Elastic Beanstalk - Supported Platforms

- Go
- Java SE
- Java with Tomcat
- .NET Core on Linux
- .NET on Windows Server
- Node.js
- PHP
- Python
- Ruby
- Packer Builder
- Single Container Docker
- Multi-container Docker
- Preconfigured Docker

#### Web Server Tier vs Worker Tier

- Web Server Tier

  - loadbalancer가 있고 loadbalancer가 트래픽을 auto scaling group에 전송
  - auto scaling group에는 웹 서버가 될 다수의 EC2 인스턴스 존재

- Worker Tier
  - EC2 인스턴스에 직접적으로 액세스하는 클라이언트가 없음
  - 메세지 대기열인 SQS Queue를 사용할 것이고 메세지는 SQS Queue로 전송
  - EC2 인스턴스가 worker가 됨
    - EC2 인스턴스가 메세지를 SQS Queue에 가져와 처리하기 때문
  - Worker 환경은 SQS 메세지의 개수에 따라 스케일링
    - 메세지가 많으면 EC2 인스턴스가 많아짐
  - 웹 환경이 메세지를 워커 환경의 SQS Queue에 푸시하게 함으로써 웹 환경과 워커 환경을 하나로 모을 수 있음

#### Elastic Beanstalk Deployment Modes

- Single Instance
  - Great for dev
  - 하나의 EC2 인스턴스를 가지고 그게 탄력적 IP를 가짐
  - RDS 데이터베이스를 시작할 수도 있지만 모두 탄력적 IP를 가진 하나의 인스턴스를 기반으로 함
- High Availability with Load Balancer
  - Great for prod
  - 실제 Elastic Beanstalk 모드로 스케일링하고 싶은 경우, Loadbalancer를 사용하는 고가용성(High availability) 선택
  - 로드밸런서가 있어 다수의 EC2 인스턴스에 분산시킬 수 있으며 그것이 auto scaling group과 multi availability zone에 맞춰 관리
  - Multi Availability zone이고 Standby가 있는 RDS DB도 가질 수 있음

## Elastic Beanstalk 실습

#### Elastic Beanstalk 콘솔

- Create application

  - Environment tier : Web Server Environment
    - 웹사이트를 실행하길 원하면 Web Server Environment, Process task를 처리하려면 Worker Environment
  - Application name: MyApplication
  - Environment name: MyApplication-dev로 변경
  - Platform
    - Platform Type: Managed platform
    - Platform: Node.js
  - Application code: Sample application
  - Presets: Single instance
    - 고가용성을 쓰고 싶으면 High availability, 커스텀하고 싶으면 Custom configuration
  - NEXT
  - Service Access
    - Beanstalk이 해야할 일을 할 수 있게 해 줄 IAM 역할
    - EC2 instance profile 채우기
      - Role 생성
      - IAM>Roles> Truested entity type: AWS service, Use case: EC2 설정, Permissions policies에서 `beanstalk` 입력 후 Web tier, Worker tier, MulticontainerDocker 선택, Role name: aws-elasticbeanstalk-ec2-role
    - Beanstalk로 돌아와 EC2 instance profile에 방금 생성한 role 선택
  - NEXT 후 선택사항은 모두 Skip할 것이므로 Skip to review
    - single 인스턴스 모드에 설정한 기본값 사용할 것
  - Submit으로 Beanstalk 환경 생성

- CloudFormation 콘솔에서 Beanstalk 생성 확인

  - CloudFormation > Stacks > 생성한 Beanstalk 선택
    - Events에서 cloudFormation 템플렛이 어떻게 생성되는지 조회 가능
      - security group과 elasticIP, 인스턴스 확인 가능
    - Template에서 View in Designer 누르면 실제 cloudFormation이 생성한 아키텍처 확인 가능
    - Beanstalk에서 생성하려는 모든 리소스는 cloudFormation이 생성

- 모든 리소스가 실행되면 Beanstalk Health는 OK이고 Domain name을 얻음
  - Domain name을 클릭하고 새 탭에서 열면 Beanstalk 환경과 EC2 인스턴스에 액세스하게 됨
  - 해당 환경에 액세스가 가능하다는 것을 통해 Beanstalk이 샘플 코드로부터 내 애플리케이션과 웹 서버를 성공적으로 시작하기 위한 모든 인프라를 생성하였음을 알 수 있음

- 새로운 버전을 업로드하고 싶은 경우
    - Elastic Beanstalk> Environments > MyApplication-dev에서 Upload and deploy 클릭
    - 새로운 application을 upload하고 deploy함으로써 EC2 인스턴스에 자동으로 배포

- Elastic Beanstalk> Environments > MyApplication-dev의 Managed updates는 Beanstalk이 환경을 업데이트할 때 나타남

- 좌측의 Configuration을 클릭하면 Beanstalk 환경에 관한 모든 설정을 조회, 수정, 적용할 수 있음

> Beanstalk은 코드와 코드를 위한 환경 중심이고, CloudFormation은 다양한 종류의 인프라로 stalk을 임의로 배포하는 데 사용됨