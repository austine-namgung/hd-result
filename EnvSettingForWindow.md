
# 1. 설치 프로그램
## 1. Docker Desktop
- https://www.docker.com/products/docker-desktop - Docker 설치 파일
- https://docs.microsoft.com/ko-kr/windows/wsl/wsl2-kernel - 윈도우 Docker 에서는 wsl 2 가 설치 되어있어야합니다.
## 2. Github Desktop
- 소스 형상관리 툴 ( SourceTree를 써도 무방합니다. )
- https://desktop.github.com/
## 3. VSCode
- 개발 툴 ( Intellij, Eclipse 등을 사용해도 무방합니다. )
- 다양한 Extentions을 지원하여 개발편의성이 향상됩니다. 
- 설치한 Extention
* Spring Boot Dashboard
* Spring Boot Extension Pack
* Spring Boot Support
* Spring Boot Tools
* Spring Initializr Java Support
* REST Client
* Spelling and Grammar Checker 
* Auto-Open Markdown Preview 
* markdownlint 
* Lombok Annotaions Support


## 4. JDK 11
- https://jdk.java.net/archive/
- oracle jdk 11 설치
## 5. Gitbash 
- 윈도우에서 리눅스 터미널 환경을 사용하기위한 Bash Tool
- gitfowindows.org
- profile 설정
    ```
    alias k=kubectl
    JAVA_HOME=/c/jdk-11.0.2
    PATH=$PATH:$JAVA_HOME/bin
    export PATH=$PATH:/c/Users/admin/AppData/Local/Programs/Microsoft\ VS\ Code/:/c/workspace/
    ```
## 6. Dbeaver
- Database Tool
- https://dbeaver.io/download/
- Mysql 8 버전 이상에서는 접속할때 allowPublicKeyRetrieval 을 True 로 세팅해야함
## 7. Mysql For Docker
- Database
    ```
    docker search mysql
    docker pull mysql
    docker run -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=password --name mysql mysql
    ```
- root 계정으로 접속후 유저생성, 권한 추가
    ```
    create user 'demo_common'@'localhost' identified by '1234';
    create user 'demo'@'localhost' identified by '1234';

    grant all privileges on *.* to 'demo_common'@'localhost';
    grant all privileges on *.* to 'demo'@'localhost';

    flush privileges;
    ```
## 8. Redis For Docker
- redis
    ```
    docker pull redis
    docker run --name redis -d -p 6379:6379 redis
    docker run --name redis -d -p 6379:6379 --network redis-net redis
    winpty docker run -it --network redis-net --rm redis redis-cli -h redis
    ```
- 윈도우에서 Docker run을 실행하기위해서는 앞에 <b>winpty</b> 를 붙여줘야함
## 9. Kubernetes Kind
- Docker Destop에서 Kubernetes enable 버튼을 눌러 설치
- Kind 설치  https://kind.sigs.k8s.io/docs/user/quick-start/
- gitbash에서 사용하기위해 path 설정 
## 10. Skaffold
- Dockerizing 과 배포를 같이 해주는 툴
- https://skaffold.dev/docs/install/
- gitbash에서 사용하기위해 path 설정

## 11. Nginx
- 테스트 웹페이지를 띄우기위한 웹서버
- http://nginx.org/en/download.html

## 12. Kong
- API gateway
```
docker network create kong-net
```
```
 docker run -d --name kong-database \
               --network=kong-net \
               -p 5432:5432 \
               -e "POSTGRES_USER=kong" \
               -e "POSTGRES_DB=kong" \
               -e "POSTGRES_PASSWORD=kong" \
               postgres:9.6
```
```
docker run --rm \
     --network=kong-net \
     -e "KONG_DATABASE=postgres" \
     -e "KONG_PG_HOST=kong-database" \
     -e "KONG_PG_USER=kong" \
     -e "KONG_PG_PASSWORD=kong" \
     -e "KONG_CASSANDRA_CONTACT_POINTS=kong-database" \
     kong:latest kong migrations bootstrap
```
```
docker run -d --name kong \
     --network=kong-net \
     -e "KONG_DATABASE=postgres" \
     -e "KONG_PG_HOST=kong-database" \
     -e "KONG_PG_USER=kong" \
     -e "KONG_PG_PASSWORD=kong" \
     -e "KONG_CASSANDRA_CONTACT_POINTS=kong-database" \
     -e "KONG_ADMIN_LISTEN=0.0.0.0:8001, 0.0.0.0:8444 ssl" \
     -p 8000:8000 \
     -p 8443:8443 \
     -p 127.0.0.1:8001:8001 \
     -p 127.0.0.1:8444:8444 \
     kong:latest
```
## 13. Konga
- Kong Admin 을 쉽게 사용하기위한 UI
```
docker pull pantsel/konga
docker run -p 1337:1337 \
             --network kong-net \
             --name konga \
             -e "NODE_ENV=production" \
             -e "TOKEN_SECRET=token" \
             pantsel/konga
```

# 2. 프로젝트 실행
<b>윈도우 환경에서는 9001 번 포트를 Intel(R) Graphics Command Center 에서 사용하고 있기때문에 해당서비스를 먼저 종료해야함 </b>

## 1. API 인증
- common-api, vehicle-api, vehicle-fe 프로젝트를 vscode로 오픈
- vehicle-fe (kong/kong2 branch), common-api,vehicle-api (kong branch)
- 터미널에서 각각 아래 명령어 입력
    ```
    skaffold dev -p kind --port-forward
    ``` 
- common-api 프로젝트의 ~/kong/kong_setting.json 파일을 Import 
- ngnix root 경로를 web 경로로 변경
(vehicle-fe 프로젝트의 테스트 웹페이지경로로 설정)

