### EC2
- 인스턴스, 서버

### 사용법
- EC2 -> 인스턴스 생성 -> 보안키 생성 -> 실행
- 인스턴스 -> 보안 -> 인바운드 규칙 (설정 안하고 생성 시 해줘야함, Default SSH)
  - HTTP
  - HTTPS

### --
- 퍼블릭 IP는 변경 됨, 프라이빗은 고정 값, 퍼블릭 IP 고정 값 하기 위해선 Elastics IP 생성 후 associate IP에서 인스턴스 등록


### 보안규칙과 포트
- 보안 규칙은 여러 개 만들 수 있고, 하나의 인스턴스에 여러 개 혹은 하나의 보안 규칙을 할당할 수 있다.
- time out -> 보안 규칙 이슈
- connection refuse -> ec2 내부 이슈
- 모든 inbound 는 default로 막혀 있기 때문에 열어 줘야 하고, outbound는 열려있기에 필요 시 변경

### 터미널 접속
1. 인스턴스에서 connect
2. SSH 접속
   - 폴더 생성 후 pem 파일 이동 chmod 400 newkeypair.pem
   - ssh -i newkeypair.pem ubuntu(username)@PublicIP

### Elastic Block Store (EBS)
- 인스턴스를 백업하기 위한 기능
- USB라고 생각하면 된다. 인스턴스 삭제해도 백업시켜뒀다면 데이터가 남는다.
- EBS Snapshot을 통해 날짜 별 EBS를 관리할 수 있다.
- EBS는 현재 AZ, EBS Snapshot은 다른 AZ도 가능


### Volume 사용법
- EBS -> 볼륨 -> 볼륨 생성 -> 생성 후 생성된 볼륨에서 작업, 연결
- 기존 인스턴스와 연결 시 AZ 같게 해야 하고 원래 연결된 볼륨이 있다면 추가로 연결된다.
- Delete Option Default No이기에 인스턴스 삭제 시 볼륨은 남는다. (인스턴스 생성 시에도 볼륨 설정은 가능)

### Snapshot 사용법
- EBS -> 볼륨 -> 작업, 스냅샷 생성
- 스냅샷 내에서도 볼륨을 만들 수 있다. 복사를 통해 다른 AZ로 이동할 수 있다.

## Amazon Machine Image
- 운영체제 이미지 파일, 커스텀 이미지 생성 가능
- 공유 가능

### 사용법
- 인스턴스에서 Create Image -> AMI 확인 후 생성된 것 확인 (유저 네임 ubuntu에서 root로 변경되기에 SSH 접속 시 확인)