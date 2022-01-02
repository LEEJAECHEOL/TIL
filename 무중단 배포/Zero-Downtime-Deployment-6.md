# 무중단 배포 - 로그백업

```sh
DATE=`date +%Y%m%d_%H%M%S`
FROM_PATH=/home/ubuntu/nonstop/app
TO_PATH=/home/ubuntu/nonstop/backup/log
PREV_DATE=1

cp $FROM_PATH/log.out $TO_PATH/log_backup_$DATE.out
cat /dev/null > $FROM_PATH/log.out
tar -cvf $TO_PATH/log_backup_$DATE.tar $TO_PATH/log_backup_$DATE.out
/usr/bin/rm $TO_PATH/log_backup_$DATE.out

/usr/bin/find $PATH/ -mtime +$PREV_DATE -delete
```

- 하루에 한번 씩 백업을 원한다면 crontab을 사용하면 됨.
- 로그는 파일 용량이 크기 떄문에 압축까지 적요함.
