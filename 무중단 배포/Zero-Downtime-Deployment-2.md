# 무중단 배포 - 배포 스크립트

- /app/deploy.sh

```sh
BASE_PATH=/home/ubuntu/app/nonstop
BUILD_PATH=$(ls $BASE_PATH/*-SNAPSHOT.jar)
JAR_NAME=$(basename $BUILD_PATH)
echo ">>> Check Profile that is current running Application"
CURRENT_PROFILE=$(curl -s http://localhost/profile)
echo ">>> Running Application Profile : $CURRENT_PROFILE"

if [ $CURRENT_PROFILE == env1 ]
then
        IDLE_PROFILE=env2
        IDLE_PORT=8082
elif [ $CURRENT_PROFILE == env2 ]
then
        IDLE_PROFILE=env1
        IDLE_PORT=8081
else
        echo ">>> NOT FOUND PROFILE : $CURRENT_PROFILE"
        echo ">>> set env1  : PROFILE: env1"
        IDLE_PROFILE=env1
        IDLE_PORT=8081
fi

echo ">>> Check PID  Running $IDLE_PROFILE Application"
IDLE_PID=$(pgrep -f active=$IDLE_PROFILE)
echo ">>> PID : $IDLE_PID"

if [ -z $IDLE_PID ]
then
        echo ">>> NO Running Application"
else
        echo ">>> kill -15 $IDLE_PID"
        sudo kill -15 $IDLE_PID
        echo ">>> Deploy start in 10 seconds"
        sleep 10
fi


echo ">>> Deploy $IDLE_PROFILE"
sudo nohup java -jar -Dspring.profiles.active=$IDLE_PROFILE $BASE_PATH/$JAR_NAME &
echo ">>> Start application($IDLE_PROFILE) health check in 10 seconds"
echo ">>> curl -s http://localhost:$IDLE_PORT/actuator/health"
sleep 10

for retry_count in { 1..10 }
do
        response=$(curl -s http://localhost:$IDLE_PORT/actuator/health)
        echo ">>> response : $response"
        up_count=$(echo $response | grep 'UP' | wc -l)
        echo ">>> up_count : $up_count"

        if [ $up_count -ge 1 ]
        then
                echo ">>> Health Check Success!!"
                break
        else
                echo ">>> Unknown Health Check Response or Status is not 'UP'"
                echo ">>> Health Check : $(response)"
        fi

        if [ $retry_count -eq 10 ]
        then
                echo ">>> Fail Health Check!"
                echo ">>> End the deploy whithout connect to NGINX"
                exit 1
        fi

        echo ">>> Fail Healt Check!! Retry..."
        sleep 10
done
```

- 실행 권한 주기

```
chmod +x deploy.sh
```

- 흐름

  1. 현재 nginx가 가리키는 스프링의 profile을 확인 후, 가리키지 않는 포트를 IDLE_PORT로 지정한다
  2. pgrep -f active=$IDLE_PROFILE 명령어를 통해 구동 중인 어플리케이션이 있다면 PID를 가져온다
  3. 구동 중인 어플리케이션을 중지한다
  4. 새로운 어플리케이션을 가동한다.
  5. 10초 후 어플리케이션의 health 체크를 수행한다.
  6. 10번을 실패하면 exit 1로 빠져나가며, 성공 반복문을 빠져나가고 switch.sh를 실행한다.

- Error
  - Jenkins 사용 도중 Build Shell 입력작업중, if 구문에서 에러가 계속 발생하는 경우
  - sh -> dash 로 링크가 걸려있어서 발생한 문제
  - root 계정으로 들어가서 /bin/sh 링크를 /bin/bash 로 재설정
    > ### 현재 상태 확인 방법
    >
    > - ls -ahl /bin/sh
    >
    > ### 링크 해제 후, 재설정
    >
    > - unlink /bin/sh
    > - ln -s /bin/bash /bin/sh
