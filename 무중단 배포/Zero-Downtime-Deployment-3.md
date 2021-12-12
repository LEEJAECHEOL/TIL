# 무중단 배포 - 전환 스크립트

- nginx가 가리키는 스프링 서버를 바꾸는 스크립트
- /app/switch.sh

```sh
echo ">>> Current Running PORT"
CURRENT_PROFILE=$(curl -s http://localhost/profile)

if [ $CURRENT_PROFILE == env1 ]
then
        IDLE_PORT=8082
elif [ $CURRENT_PROFILE == env2 ]
then
        IDLE_PORT=8081
else
        echo ">>> Not Found Profile : $CURRENT_PROFILE"
        echo ">>> set PORT 8081"
        IDLE_PORT=8081
fi

PROXY_PORT=$(curl -s http://localhost/profile)
echo ">>> Current Running Profile : $CURRENT_PROFILE"

echo ">>> Change Port : $IDLE_PORT"
echo "set \$service_url http://127.0.0.1:${IDLE_PORT};" | sudo tee /etc/nginx/conf.d/service-url.inc

echo ">>> NGINX RELOAD"
sudo service nginx reload
```

- 실행 권한 주기

```
chmod +x switch.sh
```

- 흐름
  1. nginx가 구동 중인 프로필을 확인 후, 새롭게 가리킬 포트를 IDLE_PORT에 설정
  2. /etc/nginx/conf.d/service-url.inc 파일에 포트 부분을 IDLE_PORT로 변경
  3. nginx가 새로 바꾼 설정을 읽도록 리로드
