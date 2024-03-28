## AWS Cloud Use
- 복잡하고 확장 가능한 애플리케이션 생성 가능
- 다양한 산업 분야에 적용 가능
    - 기업 IT 담당, 백업과 저장소 제공, 빅데이터 분석
    - 웹사이트 호스팅, 모바일&소셜 앱에 대한 백엔드
    - 게임 서버 전체가 클라우드에서 실행되도록 가능

## AWS Global Infrastructure
- AWS Regions
- AWS Availability Zones
- AWS Data Centers
- AWS Edge Locations / Points of Presence

참고: <https://infrastructure.aws/>

#### AWS Regions

- Region은 전세계에 걸쳐있고 이름 존재
    - region name: us-east-1, eu-west-3 등
- Region은 `데이터 센터의 집합`
- `대부분의 AWS 서비스는 특정 region에 연결되어 국한됨`

- 알맞은 region 선택할 때 고려해야 하는 요소: 새로운 애플리케이션을 시작할 때 어느 region을 선택하는 것이 좋은가?
    - `Compliance` (법률 준수)
        - 어떤 정부는 애플리케이션을 배포하게 될 대상 국가 내에 데이터가 보관되기를 원함
        - Ex. 프랑스의 데이터는 프랑스 내에 존재해야함
    - `Proximity` (지연 시간)
        - 지연 시간을 줄이는 방향으로 선택
        - 대부분의 사용자가 미국에 있는 경우 Application도 사용자와 가까이 있도록 미국에서 출시하는 것이 바람직
    - `Available services` (가능한 서비스)
        - 모든 region이 모든 서비스를 가지지 않음
        - 애플리케이션이 특정 서비스를 활용해야한다면 해당 region에서 해당 서비스를 사용할 수 있는지 확인
    - `Pricing` (가격)
        - region마다 가격이 다름

#### AWS Availability Zones
- 각각의 region은 많은 availability zone을 가짐
    - 보통 3개를 갖고, 최소 3개 최대 6개를 가질 수 있음
    - Ex. Sydney region은 3개의 Availability zone을 가짐 => ap-southeast-2a, ap-southeast-2b, ap-southeast-2c

- 각각의 Availability zone(AZ)은 많은(여분의) 전원, 네트워킹과 통신 기능을 갖춘 하나 또는 두 개의 개별적인 데이터 센터로 이루어짐
- 각각의 AZ이 재난 발생에 대비해 서로 분리되어 있음
    - 즉 ap-southeast-2a에 문제가 발생해도, ap-southeast-2b와 ap-southeast-2c는 영향을 미치지 않음

- AZ와 AZ 내의 데이터 센터는 높은 대역폭의 초저지연 네트워킹으로 서로 연결되어 Region 형성

#### AWS Points of Presence :Edge Locations
- global infra로서 AWS에 대해 꼭 알고 있어야하는 점 => `Edge Location`

- AWS가 40개 이상의 국가의 90개 이상의 도시에 400개가 넘는 전송 지점 (Edge location)을 가짐
- 이로 인해 최종 사용자에게 컨텐츠를 최소 지연 시간으로 전달할 수 있음

## AWS Console

- AWS의 Global Service
    - IAM (Identity and Access Management)
    - DNS service (Route53)
    - CloudFront (Content Delivery Network)
    - WAF (Web Application Firewall)

- AWS의 Region-scoped Service
    - Amazon EC2 (Infrastructure as a Service)
    - Elastic Beanstalk (Platform as a Service)
    - Lambda (Function as a Service)
    - Rekognition (Software as a Service)

- Region Table
    - 특정 region에서 사용할 수 있는 서비스
    - <https://aws.amazon.com/about-aws/global-infrastructure/regional-product-services>