# Kong 설정

## 1. 개요
- Kong은 Cloud-Native 이며 빠르고 확장 가능 마이크로서비스추상화 계층 (API Gateway) 입니다.
- Kong은 플러그인을 통해로드 밸런싱, 로깅, 인증, 속도 제한, 변환 등을 제공하면서 마이크로 서비스 요청에 대한 게이트웨이 역할을 수행 합니다.
![!](./Kong1.png "")
##2. 설정방법
- 설치는 Docker 기반으로 진행되었으며
https://docs.konghq.com/install/docker/ 에서 Guide를 참고할수있습니다.
- Docker 가 미리 설치 되어있어야 합니다.
## 2. Kong 설치
1. Docker Network 만들기

    ```shell script
    docker network create kong-net
    ```

2. Database 만들기 (둘중하나 만들기)
   
    - Cassandra
    ```shell script 
    docker run -d --name kong-database \
                   --network=kong-net \
                   -p 9042:9042 \
                   cassandra:3
    ```
     - PostgreSQL (여기에서는 PostgreSQL 사용)
    ```shell script 
        docker run -d --name kong-database \
                       --network=kong-net \
                       -p 5432:5432 \
                       -e "POSTGRES_USER=kong" \
                       -e "POSTGRES_DB=kong" \
                       -e "POSTGRES_PASSWORD=kong" \
                       postgres:9.6
    ```
   
3. Database Migration
    ```shell script 
    docker run --rm \
         --network=kong-net \
         -e "KONG_DATABASE=postgres" \
         -e "KONG_PG_HOST=kong-database" \
         -e "KONG_PG_USER=kong" \
         -e "KONG_PG_PASSWORD=kong" \
         -e "KONG_CASSANDRA_CONTACT_POINTS=kong-database" \
         kong:latest kong migrations bootstrap
    ```
4. Kong 시작
    ```
   docker run -d --name kong \
        --network=kong-net \
        -e "KONG_DATABASE=postgres" \
        -e "KONG_PG_HOST=kong-database" \
        -e "KONG_PG_USER=kong" \
        -e "KONG_PG_PASSWORD=kong" \
        -e "KONG_CASSANDRA_CONTACT_POINTS=kong-database" \
        -e "KONG_PROXY_ACCESS_LOG=/dev/stdout" \
        -e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" \
        -e "KONG_PROXY_ERROR_LOG=/dev/stderr" \
        -e "KONG_ADMIN_ERROR_LOG=/dev/stderr" \
        -e "KONG_ADMIN_LISTEN=0.0.0.0:8001, 0.0.0.0:8444 ssl" \
        -p 8000:8000 \
        -p 8443:8443 \
        -p 127.0.0.1:8001:8001 \
        -p 127.0.0.1:8444:8444 \
        kong:latest
    ```
    - 8001 번 포트는 Admin API 포트이며 8001번 포트의 호출을 통해 설정을 변경할수있습니다.
    - 8000 번 포트는 Kong 기본 포트입니다.
5. Konga 
    1. Database migration
        ```
       docker run --rm --network=kong-net pantsel/konga:latest -c prepare -a 'postgres' -u postgresql://kong:kong@kong-database:5432/konga
        ```
   2. Konga 시작
       ```
       docker run -p 1337:1337 --network=kong-net -e "TOKEN_SECRET=YOUR_TOKEN_SECRET" -e "DB_ADAPTER=postgres" -e "DB_URI=postgresql://kong:kong@kong-database:5432/konga" -e "NODE_ENV=production" --name konga pantsel/konga
       ```
   3. Admin 등록
       ![!](./Kong2.png "")
       http://localhost:1337/ 로 접속하여 Konga 처음실행시 Admin 정보를 입력해야합니다.
   4. Kong 연동
       ![!](./Kong3.png "")
       Konga 과 연결할 Kong 의 정보를 입력합니다.

## 3. 결과화면
![!](./Kong4.png "")
설치완료시 http://localhost:1337/ 로 들어가서 다음과 같은 화면을 볼수있습니다.

## 4. 참조
- https://konghq.com/
- https://pantsel.github.io/konga/




