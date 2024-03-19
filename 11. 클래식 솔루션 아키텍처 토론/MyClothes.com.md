## Stateful Web App:MyClothes.com

- 이전 서비스는 DB가 필요없기에 stateless한 상태의 서비스였지만, 본 서비스는 DB가 필요하지만 최대한 Stateless하게 진행해야함 

#### Service 설명

- 사람들이 온라인으로 옷을 살 수 있게 해줌
- 장바구니 존재
- 수백명의 사용자가 있고 모든 사람들이 동시에 웹사이트를 둘러봄
- 확장할 수 있어야하고 수평 확장성을 유지하며 application을 최대한 stateless하게 유지하고 싶음
    - 사용자가 장바구니를 잃어버리면 안 된다는 뜻
- 주소 등의 사용자 정보를 효과적으로 보관하고 어디에서나 접근할 수 있는 데이터 베이스에 저장

#### Service 기본 구조

- User
- Route53
- Multi AZ ELB
- Auto Scaling group과 3개의 AZ에 Instance 존재

#### Introduce Stickiness (Session Affinity)

> User 입장: Application이 ELB에 접근하고 ELB는 Instance1을 가리킴. 다음 요청을 하면 ELB는 이전과 다른 Instance2를 가리킴. 이때 장바구니 정보가 사라짐

- `ELB Stickiness 활성화`
    - ELB Stickiness에 의해 동일한 User가 하는 모든 요청은 동일한 Instance로 가게 됨

#### Introduce User Cookies

> Stickiness를 사용하고 있어도 EC2 Instance가 어떤 이유로든 종료되면 장바구니를 잃어버리게 됨

- `User Cookies` 사용
    - 기본적으로 EC2 Instance가 장바구니 내용을 저장하는 대신, User 쪽에서 장바구니 내용을 `Web Cookies`를 통해 저장하도록 함
    - User가 ELB에 접속하여 EC2 Instance로 갈 때마다 자신의 장바구니 정보를 보여줌
    - 각각의 EC2 Instance가 이전의 일을 알 필요 없는 Stateless한 상태 달성

> EC2는 stateless해졌지만 User가 전달할 내용이 존재해 HTTP request가 점점 더 무거워지는 문제 발생 + Cookie가 공격받아 장바구니가 수정될 수 있는 보안 위험 존재  

- 따라서 EC2 Instance가 반드시 User의 cookie를 검증해야하며 전체 Cookie의 크기는 4KB 이하만 가능

#### Introduce Server Session

> User Cookie는 전송할 수 있는 용량이 매우 적고 보안이 불안정

- `Server Seccion` 사용
    - User가 전체 장바구니를 Web Cookie로 보내는 대신 단순히 Session ID만 보내는 것
        - User에 대한 Session ID
    - 백그라운드에는 `ElastiCache` 클러스터 존재
    - User가 Web Cookie로 가지고 있는 Session ID와 요청 사항을 ELB에  함께 전송 > ELB는 EC2 Instance에 Session ID와 요청을 보내면서 장바구니에 특정 물건을 추가(요청)하라고 보냄 > EC2 Instance는 장바구니 내용을 ElastiCache에 추가
        - 이 장바구니 내용을 불러올 수 있는 ID가 Seccion ID가 됨
        - EC2 Instance는 session ID를 통해 ElastiCache로부터 특정 사용자의 장바구니를 찾고 session data 불러옴
    - ElastiCache는 성능이 매우 좋다는 장점도 존재
    - session data 저장의 또다른 방식으로 Amazon DynamoDB도 존재

#### Storing User Data in a database

> 사용자 주소 등의 User 데이터를 DB에 저장하고 싶음

- `RDS Instance`
    - EC2 Instance가 RDS Instance와 통신 
    - RDS는 장기적인 저장을 위한 DB로 좋으며 직접 통신함으로써 주소, 이름 등의 사용자 데이터를 저장하거나 불러올 수 있음
    - 각각의 Instance가 RDS와 통신할 수 있으며 일종의 Multi AZ Stateless 솔루션을 얻을 수 있음

#### Scaling Reads

> traffic이 늘어났는데 대부분의 사용자가 웹사이트를 둘러보며 시간을 보냄

- writes를 수행하는 `RDS Master` 사용
    - RDS DB 읽기 확장을 위해 writes RDS Master 복제해 RDS Read 복제본 사용
        - RDS는 5개의 읽기 전용 복제본을 가질 수 있음
    - User가 무언가를 읽을 때 읽기 전용 복제본인 RDS Read Replicas부터 읽음

#### Scaling Reads (Alternative) - Write Through

> 위의 읽기 전용 Scaling을 해결하는 또 다른 방법

- `Cache를 사용하는 Write Mode`
    - User가 요청을 보내면 EC2는 ElastiCache에 존재하는지 확인하고 가지고 있지 않다면 RDS로부터 읽어와 ElastiCache에 저장 
        - 정보가 캐싱
    - 이 방식을 통해 RDS의 트래픽을 줄이고 RDS의 CPU 사용량을 줄여 성능을 향상시킴
    - But 캐시 유지 보수가 필요하며 application 쪽에서 이루어져야함

#### Multi AZ - Survive disasters

> 재해 가능성

- `Multi AZ` 사용
    - 현재 ELB도 Multi AZ , EC2 Instance도 Auto-Scaling group이면서 Multi AZ, RDS도 기본적으로 Multi AZ를 가짐
    - Redis를 사용하면 ElastiCache도 Multi AZ를 가질 수 있음

#### Security Groups

- `안전한 Security Groups`
    - ELB는 어디에서나 HTTP, HTTPS 트래픽을 열 수 있음 `0.0.0.0/0`
    - EC2는 ELB로부터 오는 트래픽만으로 제한
    - ElastiCache와 RDS는 EC2 security group으로부터 오는 트래픽만으로 제한

## Service 적용 개념
- ELB Sticky sessions
- Web clients for storing cookies and making our web app stateless
- ElastiCache
    - For Storing sessions (alternative: DynamoDB)
    - For caching data from RDS
    - Multi AZ
- RDS
    - For storing user data
    - Read replicas for scaling reads
    - Multi AZ for disaster recovery
- Tight Security with security groups referencing each other

## 해당 서비스는 client,web,database tier 3개의 tier가 존재하는 매우 보편적인 구조