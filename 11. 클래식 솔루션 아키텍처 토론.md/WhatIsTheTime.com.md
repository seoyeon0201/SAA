## Stateless Web App : WhatIsTheTime.com

- 시간을 알려주는 서비스
    - DB는 필요없고 각 instance와 server는 시간을 알고 있음
    - `downtime 제거`, `수직, 수평으로 확장`를 목적으로 시작

#### Starting Simple

- User
- t2.micro Public EC2 instance 

- `Elastic IP Address` (탄력적 IP 주소) 사용
    - 무슨 일이 생기면 재시작할 수 있도록 EC2 instance가 `고정 IP address를 갖도록` 

#### Scaling Vertically

- 더 많은 트래픽을 갖게 되면서 t2.micro instance로는 충분하지 않음을 깨달음
- `부하 처리`를 위해 m5.large instance로 `vertically scaling` (수직 확장)
    - instance 중지 > instance 유형 변경 > instance 재시작
        - downtime 발생 => 추후 해결
    - 위에서 탄력적 IP 설정을 했기에 동일한 공용 IP를 가지게 되고 사용자 접근 가능

#### Scaling Horizontally

- M5 application은 하나의 공용 IP를 가지고 있는데, 정말 많은 사용자가 접근하기엔 힘들기에 `horizontally scaling` (수평확장) 필요
    - m5.large 유형의 EC2 instance 추가
    - 이 instance는 모두 각각의 탄력적 IP 연결
        - 사용자가 3개의 정확한 탄력적 IP를 알아야 함

- 위 방식은 사용자가 더 많은 탄력적 IP를 알아야하고 관리자는 더 많은 인프라를 관리해야하므로 적절하지 않음
    - 탄력적 IP는 한 region마다 5개의 탄력적 IP만을 가질 수 있으므로 적절하지 않음

- `탄력적 IP 제거`하고 `Route53 사용`
    - 웹사이트 URL은 api.whatisthetime.com
    - A record
        - Route53의 A record는 IP를 의미
    - TTL은 1 hour

#### Scaling horizontally, adding and removing instances

- 상황에 따라 `인스턴스를 추가하고 제거할 수 있도록 확장하고 싶음` 
    - 해당 instance IP를 cache로 저장한 사용자는 TTL에서 남은 시간만큼 down time을 가지게 됨

#### Scaling horizontally, with a load balancer

- Public EC2 instance를 Private EC2 instance로 변경
    - 모두 같은 Availability zone(가용영역)에서 실행
    - load balancer도 동일한 Availability zone에 존재

- `load balancer` 사용
    - ELB는 health check 기능이 있어 instance가 다운되거나 작동하지 않으면 사용자로부터 해당 instance로 트래픽을 전송하지 않음
    - ELB와 Private EC2 instance를 연결하면 ELB는 공개되겠지만 instance는 숨어있음
    - security group rule을 이용해 트래픽 제한
- `Alias record` 사용
    - load balancer가 IP address를 지속적으로 바꾸기 때문에 A record가 아닌 Alias record 사용
    - User는 Route53을 통해 ELB로 접근하고 ELB는 health check를 통해 healthy instance로만 트래픽 전송

#### Scaling horizontally, with an auto-scaling group

- 수동으로 인스턴스를 추가하고 제거하는 것은 어려움 => `Auto-scaling group` 실행
    - Private instance를 Auto Scaling group이 관리
    - 기본적으로 auto-scaling group이 요청에 따라 확장,축소하도록 함
    - auto-scaling과 loadbalancing이 있으니 down time이 없음

#### Making our app multi-AZ

- 자연재해가 발생하는 경우, Availability zone이 다운되어 applicatoin 다운 => 가용성을 높이기 위해 "multi-AZ" 추천

- ELB에 `Multi AZ` 사용
    - Availability zone 1부터 3에서 실행되며 이 ELB는 3개의 AZ가 존재
    - auto-scaling group도 Multi AZ에 걸쳐 있음
        - Ex. Availability zone 1이 다운되어도 2,3가 있기 때문에 트래픽 처리가 가능

#### Minimum 2 AZ 

- 위의 상황에서 최소 2개의 AZ가 존재하고 각각의 AZ에는 최소 하나 이상의 인스턴스가 실행 중 => `Reserve Capacity`
    - 기본적으로 애플리케이션의 비용을 줄이는 작업
    - 1년 내내 2개의 instance가 항상 실행 중이기 때문
    - auto-scaling group의 capacity를 최소화하기 위해 인스턴스를 reserve함으로써 미래에 상당한 비용을 절감할 수 있음
    - 극단적으로 비용 절감을 위해 spot instance를 사용할 수 있음

#### 서비스 적용 개념

- Public vs Private IP and EC2 instances
- Elastic IP vs Route53 vs Load Balancers
- Route53 TTl, A records and Alias Records
- Maintaining EC2 instances manually vs Auto Scaling Groups
- Multi AZ to survive disasters
- ELB Health Checks
- Security Group Rules
- Reservation of capacity for costing savings when possible

#### Solution Architecture 고려 요소

`costs`, `performance`, `reliability`, `security`, `operational excellence`