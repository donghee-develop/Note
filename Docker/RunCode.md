# 도커 컨테이너 생성 명령어

- mysql : docker run -d --name 이름 -p 3306:3306 -e MYSQL_ROOT_PASSWORD=비밀번호 -v mysql-data:/var/lib/mysql mysql:8.0.35
- redis : docker run -d --name 이름 -p 6379:6379 redis:alpine


# 오류 코드
1. docker: Error response from daemon: Conflict. The container name "/soketTest" is already in use by container "4c99718ed7ed44dade1992dd1b5e6c9ba430696ac940028d0606427fec485989". You have to remove (or rename) that container to be able to reuse that name.
    - 이미 있는 컨테이너

2. docker: Cannot connect to the Docker daemon at unix:///Users/donghui/.docker/run/docker.sock. Is the docker daemon running?.
    - 도커 컨테이너가 꺼져 있음, 앱 실행