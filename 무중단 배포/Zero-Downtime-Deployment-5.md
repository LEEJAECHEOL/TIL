# 무중단 배포 - DB백업

```sh
DATE=`date +%Y%m%d_%H%M%S`
PATH=/home/ubuntu/nonstop/backup/db
PREV_DATE=1

USERNAME=test
PASSWORD=1q2w3e4r
DATABASE=test

/usr/bin/mysqldump -u$USERNAME -p$PASSWORD $DATABASE > $PATH/mysql_backup_$DATE.sql
/usr/bin/find $PATH/ -mtime +$PREV_DATE -delete
```

- 하루에 한번 씩 백업을 원한다면 crontab을 사용하면 됨.
