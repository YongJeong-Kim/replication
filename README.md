# replication

```bash
$ docker pull mysql
$ docker compose up -d
```

docker compose의 volumes를 사용한 my.cnf 수정은 제대로 동작하지 않는다.

#### my.cnf를 container 내부로 복사하기
```bash
$ docker compose cp ./master/my.cnf master:/etc
$ docker compose cp ./slave/my.cnf slave:/etc
```

#### 재부팅하여 my.cnf 적용
```bash 
$ docker compose restart
```

#### server-id 확인
```bash
# master
mysql> select @@server_id;
>> 1

# slave
mysql> select @@server_id;
>> 2
```

master는 1, slave는 2가 조회되어야 한다.

#### 복제에 사용할 사용자 등록
```bash
$ docker compose exec master bash
$ mysql -u root -p
mysql> create user 'testsuser'@'%' identified by '1234';
mysql> alter user 'testuser'@'%' identified with mysql_native_password by '1234';
mysql> grant replication slave on *.* on 'testuser'@'%';
mysql> flush privileges;
```

#### master 상태 조회
```bash
# master
mysql> show master status\G
```

![master_status](https://github.com/YongJeong-Kim/replication/assets/30817924/790cc9db-c6ba-488b-9fb1-8fe643cfd42d)

- file과 position값을 slave에 사용한다.

이후  master에서 사용자 등록 등 실행하면 position 값이 바뀌어 slave에 등록할 position과 일치하지 않을 수 있다. slave에서 replication 설정을 마칠 때까지 master에서는 복제 사용자까지만 등록하고 다른 작업은 잠시 멈춰두자

#### slave에서 master 복제 등록하기
```bash
# slave
mysql> change master to MASTER_HOST='xxx.xxx.xxx.xxx', MASTER_USER='testuser', MASTER_PASSWORD='1234', MASTER_LOG_FILE='mysql-bin.000001', MASTER_LOG_POS=1153;
mysql> start slave;
```
- MASTER_LOG_FILE
  - master 상태조회에서 file명
- MASTER_LOG_POS
  - master 상태조회에서 position

#### slave 상태 조회하기
```bash
# slave
mysql> show slave status\G
```
![slave_status](https://github.com/YongJeong-Kim/replication/assets/30817924/6b415abb-8a6d-42b8-a5e1-e9c6aa96f244)
- Slave_IO_State가 Waiting for source to send event로 되어있는 것을 확인한다.

