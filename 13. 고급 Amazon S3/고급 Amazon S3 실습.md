## S3 수명 주기 규칙 실습

> S3 > Buckets > 원하는 Bucket 선택

- Bucket Lifecycle Rule 생성

  - Management 탭에서 Create lifecycle rule

    - Lifecycle name: DemoRule
    - Choose a rule scope: Apply to all objects in the bucket, 박스 체크
    - Lifecycle rule actions

      - `Move current versions of objects between storage classes`: 최신 버전 object 이동. 비저닝된 버킷 필요. Standard-IA: 30, Intelligent-Tiering:60, Glacier Instant Retrieval:90, Glacier Flexible Retrieval:180, Glacier Deep Archive:356
      - `Move noncurrent versions of objects between storage classes`: 이전 버전 이동. Glacier Flexible Retrieval:90
      - `Expire current versions of objects`: 객체 최신 버전에 만료 설정. Expire current versions of objects:700
      - `Permanently delete noncurrent versions of objects`: 이전 객체 버전 만료 설정. Permanently delete noncurrent versions of objects:700
      - `Delete expired object delete markers or incomplete multipart uploads`

    - Create Rule

## S3 이벤트 알림 실습

- bucket 생성
  - bucket name: demo-stephane-v3-event-notifications

> S3 > Buckets > 원하는 Bucket 선택

- Event 알림 설정 확인
  - Properties 탭에서 `Event notifications`과 `Amazon EventBridge` 활성화
  - 1. Amazon EventBridge: "Amazon EventBridge" Edit
    - On (활성화)
    - Save changes
  - 2. Event notifications: Ex. SQS로 전송. "Event notifications" Create event notification
    - Event name: DemoEventNotification
    - Event types: All object create events
    - Destination: SQS queue
        - 이때 Amazon SQS에서 queue 생성
            - Amazon SQS>Queues>Create queue
            - name: DemoS3Notification
            - Create queue
        - S3 bucket이 SQS queue에 작성할 수 있도록 설정
            - Access policy 탭에서 edit
            - Access policy>Policy generator
                - select policy type: SQS Queue Policy
                - Principal: *
                - Actions: SendMessage
                - ARN은 SQS Access policy의 Resource에 존재. 복붙
                - Add Statement
                - Generate Policy 클릭 후 복사
            - Access policy에 붙여넣기
            - Save
    - Event notification "Save changes"

- S3와 SQS 연결 확인
    - Amazon SQS>Queues>원하는 SQS 선택
    - Send and receive messages 선택
    - Receive messages>poll for messages 선택
    - S3로부터 테스트 메세지를 전달받은 것 확인

- Event 확인
    - Amazon S3>Buckets>원하는 bucket 선택
    - 파일 업로드
    - Amazon SQS>Queues>원하는 SQS 선택
    - 전송된 파일 확인
        - 이때 key가 coffe.jpg로 bucket에 업로드한 object임을 확인할 수 있음