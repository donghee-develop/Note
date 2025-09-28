# RDS
- 관계형 DB 서비스
## 인스턴스 내에 DB 설치 후 배포하지 않고 RDS를 사용한느 이유
- DB 인프라 자동 구축, 업데이트
- 지속 백업 복구 기능
- 모니터 대시보드 (트래픽)
- 성능 향상 read replicas
- Disaster Recovery를 위한 multi Area Zone
- 수평 수직 확장
- EBS 백업 지원
- SSH는 접속 불가능 (중반에 데이터 옮기는데 있어서 문제가 있음)

### 기능
- Storage Auto Scaling 
  - DB 용량 최대치 일 때 자동으로 늘려줌
  - Maximum Storage Threshold 설정 해줘야 함
  - 예측 불가 트래픽 떄 사용

- RDS Read Replicas
  - 서비스는 대체로 Read의 기능이 많다. 이러함에 있어 성능을 높여주고자 Read Replica (리드용 복제품)을 만들고 하나의 RDS에 대한 분산
  - 리드용이기에 SELECT 만 가능하다.

- RDS Multi AZ
  - 다른 Region으로 백업용 (고가용성을 높여준다.) + 자동 설정