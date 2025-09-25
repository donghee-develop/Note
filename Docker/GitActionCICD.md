# Git Action CI/CD
- git 내장 CI/CD, 500mb, 월 2000분 무료
- 동작 방식 : ./github/workflows 내 yaml 파일 있을 경우 자동 실행

## CI 활용
- develop merge -> gradle test
- feature/** -> gradle test
- 오류 시 메일 발송

## CI 활용 코드
```yaml
# Actions 이름 github 페이지에서 볼 수 있다.
name: Run Test

# Event Trigger 특정 액션 (Push, Pull_Request)등이 명시한 Branch에서 일어나면 동작을 수행한다.
on:
  push:
    # 배열로 여러 브랜치를 넣을 수 있다.
    branches: [ develop, feature/* ]
  # github pull request 생성시
  pull_request:
    branches:
      - develop # -로 여러 브랜치를 명시하는 것도 가능

  # 실제 어떤 작업을 실행할지에 대한 명시
jobs:
  build:
    # 스크립트 실행 환경 (OS)
    # 배열로 선언시 개수 만큼 반복해서 실행한다. ( 예제 : 1번 실행)
    runs-on: [ ubuntu-latest ]

    # 실제 실행 스크립트
    steps:
      # uses는 github actions에서 제공하는 플러그인을 실행.(git checkout 실행)
      - name: checkout
        uses: actions/checkout@v4

      # with은 plugin 파라미터 입니다. (java 17버전 셋업)
      - name: java setup
        uses: actions/setup-java@v2
        with:
          distribution: 'adopt' # See 'Supported distributions' for available options
          java-version: '17'

      - name: make executable gradlew
        run: chmod +x ./gradlew

      # run은 사용자 지정 스크립트 실행
      - name: run unittest
        run: |
          ./gradlew clean build -x test (-x 속성으로 로컬 DB 접근 안하도록)
```

## Actions (event, runner, job, step)
1. event 발생
2. runner -> job -> steps 실행 (jobs는 순차, 동시 실행이 가능하다, step은 순차적으로 진행된다.)

### event
- push, pull request 등과 같은 이벤트 발생 시 workflow 실행

### runner
- workflow 실행될 때의 instance, 각각의 job은 개별적인 runner를 통해 실행된다.

### job 
- 하나의 ruuner에서 실행될 여러 step

### step
- shell script 또는 action

### actions
- workflow의 가장 작은 단위, 재 사용 가능
- job을 만들기 위해 step들을 연결

## workflow 옵션
- name : actions의 이름
- on : 언제 실행할 지
- jobs : 실행할 내용에 대한 부분
  - runs-on : 어떤 환경에서 실행되는지
  - steps : 실제 실행할 단계
    - name : 실행에 표시될 이름
    - uses : plugin 정의
    - with : plugin에 사용할 파라미터
    - run : 실제 실행할 스크립트

## CI 실행 순서 (dev 기준)
1. dev 프로젝트 내 스크립트 생성
2. feature/test 생성 후 dev 복사, 수정 후 커밋
3. actions 확인, 실행 확인, 성공 시 Pull request
4. 풀 받아서 최신화

------------------  CD ------------------

## CD를 통해 자동 배포
- git -> gitHub -> gitHub actions -> cloud
- 흐름
  - feature/ 브랜치 push 후 PR 생성
  - PR 생성 시 gradle test 수행 (자동화)
  - PR 코드 실패 시 코드 수정하여 push 하여 PR 변경
  - 성공한 경우 승인 기다림
  - 승인 후 dev 머지 -> 주기적으로 main 머지 (main 머지 시 서버 배포)
