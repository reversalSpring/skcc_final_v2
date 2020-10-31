# part 1

## 1. Create a CDH cluster on AWS

### a. linux setup

```bash
sudo vi /etc/hosts 
 
172.31.60.204   cm.5team.com n1
172.31.51.203   master.5team.com n2
172.31.61.78    data1.5team.com n3
172.31.61.204   data2.5team.com n4
172.31.54.190   data3.5team.com n5

sudo hostnamectl set-hostname n1
```
![image](https://user-images.githubusercontent.com/30167661/97786751-08954280-1bf1-11eb-8512-38d599bb0a16.png)

```bash
ssh-keygen -t rsa
cat id_rsa.pub >> authorized_keys
```


```bash
sudo useradd training -u 3800
sudo passwd training # training
sudo groupadd skcc
sudo gpasswd -a training skcc
sudo visudo # training ALL=(ALL) ALL
```

![image](https://user-images.githubusercontent.com/30167661/97789179-b14b9e00-1c01-11eb-8f31-29d199a639d6.png)


```bash
getent hosts
cat /etc/redhat-release # CentOS Linux release 7.8.2003 (Core)
df -h
```

![image](https://user-images.githubusercontent.com/30167661/97786795-5c079080-1bf1-11eb-98dd-fce40cc5e344.png)


```bash
yum repolist
```

![image](https://user-images.githubusercontent.com/30167661/97786803-69247f80-1bf1-11eb-9d47-f026fd295764.png)



```bash
cat /etc/passwd
```

![image](https://user-images.githubusercontent.com/30167661/97786814-75104180-1bf1-11eb-9baa-7ceb20855de2.png)


```bash
cat /etc/group
```

![image](https://user-images.githubusercontent.com/30167661/97786819-7fcad680-1bf1-11eb-9d46-14647a6ea60b.png)



```bash
getent group skcc
getent passwd training
```

![image](https://user-images.githubusercontent.com/30167661/97786824-88bba800-1bf1-11eb-8de9-f7cf4194687a.png)


## 2. Mysql

### b. Install a MySQL server

```

```bash
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
transaction-isolation = READ-COMMITTED
# Disabling symbolic-links is recommended to prevent assorted security risks;
# to do so, uncomment this line:
symbolic-links = 0
# Settings user and group are ignored when systemd is used.
# If you need to run mysqld under a different user or group,
# customize your systemd unit file for mariadb according to the
# instructions in http://fedoraproject.org/wiki/Systemd

key_buffer = 16M
key_buffer_size = 32M
max_allowed_packet = 32M
thread_stack = 256K
thread_cache_size = 64
query_cache_limit = 8M
query_cache_size = 64M
query_cache_type = 1

max_connections = 550
#expire_logs_days = 10
#max_binlog_size = 100M

#log_bin should be on a disk with enough free space.
#Replace '/var/lib/mysql/mysql_binary_log' with an appropriate path for your
#system and chown the specified folder to the mysql user.
log_bin=/var/lib/mysql/mysql_binary_log

#In later versions of MariaDB, if you enable the binary log and do not set
#a server_id, MariaDB will not start. The server_id must be unique within
#the replicating group.
server_id=1

binlog_format = mixed

read_buffer_size = 2M
read_rnd_buffer_size = 16M
sort_buffer_size = 8M
join_buffer_size = 8M

# InnoDB settings
innodb_file_per_table = 1
innodb_flush_log_at_trx_commit  = 2
innodb_log_buffer_size = 64M
innodb_buffer_pool_size = 4G
innodb_thread_concurrency = 8
innodb_flush_method = O_DIRECT
innodb_log_file_size = 512M

[mysqld_safe]
log-error=/var/log/mariadb/mariadb.log
pid-file=/var/run/mariadb/mariadb.pid

#
# include all files from the config directory
#
!includedir /etc/my.cnf.d
```



```bash
sudo systemctl enable mariadb
sudo systemctl start mariadb
sudo /usr/bin/mysql_secure_installation
```

```bash
[...]
Enter current password for root (enter for none):
OK, successfully used password, moving on...
[...]
Set root password? [Y/n] Y
New password: training
Re-enter new password: training
[...]
Remove anonymous users? [Y/n] Y
[...]
Disallow root login remotely? [Y/n] N
[...]
Remove test database and access to it [Y/n] Y
[...]
Reload privilege tables now? [Y/n] Y
[...]
All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
```

```bash
hostname -f # n1.ex.com
mysql -u root -p # training

CREATE DATABASE scm DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON scm.* TO 'scm'@'%' IDENTIFIED BY 'training';
CREATE DATABASE amon DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON amon.* TO 'amon'@'%' IDENTIFIED BY 'training';
CREATE DATABASE rman DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON rman.* TO 'rman'@'%' IDENTIFIED BY 'training';
CREATE DATABASE hue DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON hue.* TO 'hue'@'%' IDENTIFIED BY 'training';
CREATE DATABASE metastore DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON metastore.* TO 'hive'@'%' IDENTIFIED BY 'training';
CREATE DATABASE sentry DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON sentry.* TO 'sentry'@'%' IDENTIFIED BY 'training';
CREATE DATABASE nav DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON nav.* TO 'nav'@'%' IDENTIFIED BY 'training';
CREATE DATABASE navms DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON navms.* TO 'navms'@'%' IDENTIFIED BY 'training';
CREATE DATABASE oozie DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON oozie.* TO 'oozie'@'%' IDENTIFIED BY 'training';
GRANT ALL ON *.* TO 'training'@'%' IDENTIFIED BY 'training';
FLUSH PRIVILEGES;

MariaDB > show databases;
```

![image](https://user-images.githubusercontent.com/30167661/97786852-b30d6580-1bf1-11eb-8838-02f34dc65678.png)


### c. Install Cloudera Manager
```bash
sudo yum install -y java-1.8.0-openjdk-devel
sudo vi /etc/profile

export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk/jre
export PATH=$PATH:$JAVA_HOME/bin
```

```bash
wget https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.49.tar.gz
tar zxvf mysql-connector-java-5.1.49.tar.gz
sudo mkdir -p /usr/share/java/
cd mysql-connector-java-5.1.49
sudo cp mysql-connector-java-5.1.49-bin.jar /usr/share/java/mysql-connector-java.jar
```

```bash
sudo vi /etc/yum.repos.d/cloudera-manager.repo

[cloudera-manager]
name = Cloudera Manager, Version 5.15.2
baseurl = https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/5.15.2/
gpgkey = https://archive.cloudera.com/redhat/cdh/RPM-GPG-KEY-cloudera
gpgcheck = 1

sudo rpm --import https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/RPM-GPG-KEY-cloudera
```

```bash
sudo yum install -y cloudera-manager-daemons cloudera-manager-server
sudo /usr/share/cmf/schema/scm_prepare_database.sh mysql scm scm training
sudo systemctl start cloudera-scm-server
```

8시 30분까지 기다려 봤으니 이 단계에서 넘어가지 않았습니다.
마지막 172.31.54.190   data3.5team.com n5 <-- 이쪽에서 안넘어 가는데
아까 sudo yum install -y cloudera-manager-daemons cloudera-manager-agent 를 철시할 때도 똑같은 172.31.54.190이 1시간 정도 잡아먹은것 같습니다.

나머지 뒷 부분은 쿼리라도 남기겠습니다.

![image](https://user-images.githubusercontent.com/30167661/97788862-629d0480-1bff-11eb-8f62-89c745092a5b.png)

[agent 설치 때도 엄청 오래 결려서 이상했떤 172.31.54.190   data3.5team.com n5] 
[밑에 사진 hostname은 다 바꾸었는데 재접속을 안해서 바뀌기 전 hostname으로 표시]
![image](https://user-images.githubusercontent.com/30167661/97788904-bf98ba80-1bff-11eb-99c3-1dc36de61691.png)



## 3. extract tables authors and posts

```bash
sqoop import \
--connect jdbc:mysql://localhost/test \
--username training \
--password training \
--table authors \
--fields-terminated-by '\t' \
--target-dir /user/training/authors

sqoop import \
--connect jdbc:mysql://localhost/test \
--username training \
--password training \
--table posts \
--fields-terminated-by '\t' \
--target-dir /user/training/posts
```

```sql
CREATE EXTERNAL TABLE authors (id int, first_name string, last_name string, email string, birthdate date, added timestamp)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
LOCATION '/user/training/authors/'

CREATE TABLE posts (id int, author_id int, title string, description string, content string, date date)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
LOCATION '/user/training/authors/'
```

# part 2
1번

```sql
SELECT a.id AS id, a.type AS type, a.status AS status, a.amount AS amount, 
a.amount - b.average AS difference FROM account AS a JOIN
(SELECT type, AVG(amount) AS average FROM account GROUP BY type) AS b
ON a.type = b.type 
WHERE status='Active';
```
![image](https://user-images.githubusercontent.com/30167661/97789022-a6443e00-1c00-11eb-8872-72d65387519b.png)


2번

```sql
CREATE DATABASE problem2;
CREATE EXTERNAL TABLE solution(id INT, fname STRING, lname STRING, address STRING, city STRING, state STRING, zip STRING, birthday STRING, hireday STRING)
STORED AS PARQUET
LOCATION '/user/training/problem2/data/employee';
```
![image](https://user-images.githubusercontent.com/30167661/97789038-c70c9380-1c00-11eb-8658-cec374691089.png)


3번

```sql
CREATE TABLE solution STORED AS PARQUET AS
SELECT c.id AS id, c.fname AS fname, c.lname AS lname, c.hphone AS hphone 
FROM account AS a JOIN customer AS c ON a.custid=c.id 
WHERE a.amount < 0;
```

![image](https://user-images.githubusercontent.com/30167661/97789053-d12e9200-1c00-11eb-8aef-f9482cf97ad2.png)


4번

```sql
CREATE DATABASE problem4;

CREATE EXTERNAL TABLE employee1(id INT, fname STRING, lname STRING, address STRING, city STRING, state STRING, zip STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' 
STORED AS TEXTFILE 
LOCATION '/user/training/problem4/data/employee1';

CREATE EXTERNAL TABLE employee2(id INT, none STRING, fname STRING, lname STRING, address STRING, city STRING, state STRING, zip STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' 
STORED AS TEXTFILE 
LOCATION '/user/training/problem4/data/employee2';

CREATE TABLE solution 
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' 
STORED AS TEXTFILE 
LOCATION '/user/training/problem4/solution' AS 
SELECT id, initcap(fname) as fname, initcap(lname), address, city, state, zip from employee1
UNION ALL
SELECT id, initcap(fname) as fname, initcap(lname), address, city, state, zip from employee2;
```

![image](https://user-images.githubusercontent.com/30167661/97789064-dc81bd80-1c00-11eb-9217-e4ff8dcc869b.png)



5번

```sql
select fname, lname, city, state from employee where city='Palo Alto' and state='CA'
union all 
select fname, lname, city, state from customer where city='Palo Alto' and state='CA';
```

![image](https://user-images.githubusercontent.com/30167661/97789069-e5728f00-1c00-11eb-9514-f6c11a684190.png)


6.

```sql
create table solution row format delimited fields terminated by '\t'
stored as textfile as
select id, fname, lname, address, city, state, zip, substr(birthday, 0, 5) from employee;
```

![image](https://user-images.githubusercontent.com/30167661/97789074-edcaca00-1c00-11eb-9bd7-ed589635ce6b.png)


7.

```sql
select concat(fname, ' ', lname) as name from employee where city='Seattle') order by name;
```
![image](https://user-images.githubusercontent.com/30167661/97789078-f6bb9b80-1c00-11eb-84fd-30935d249fe5.png)


8.

```bash
sqoop export \
--connect jdbc:mysql://localhost/problem8 \
--dirver com.mysql.jdbc.Driver \
--username cloudera \
--password cloudera \
--table solution \
-m 1 \
--export-dir '/user/training/problem8/data/customer/' \
--fields-terminated-by '\t' \
--columns 'id, fname , lname , address , city, state , zip , birthday';
```

![image](https://user-images.githubusercontent.com/30167661/97789082-fde2a980-1c00-11eb-804e-eb655440ebea.png)


9.

```sql
create table solution 
select concat('A', id) as id, fname, lname, address, city, state, zip, birthday from customer;
select * from solution;
```


![image](https://user-images.githubusercontent.com/30167661/97789088-05a24e00-1c01-11eb-8a81-a4f5c25fbc0d.png)



10

```sql
create table solution as 
SELECT c.id as id
    , c.fname as fname
    , c.lname as lname
    , c.city as city
    , c.state as state
    , b.charge as charge
    , to_date(substr(b.tstamp, 0, 10)) as billdata 
FROM customer AS c left outer join
billing AS b
ON c.id = b.id
```
![image](https://user-images.githubusercontent.com/30167661/97789097-12bf3d00-1c01-11eb-8c96-d89cd53ce844.png)

11 <-- 이거 제가 문제를 잘못 이해한걸 수도 있는데 probluem11 DB가 없습니다.

![image](https://user-images.githubusercontent.com/30167661/97789137-4f8b3400-1c01-11eb-886e-49d493615c51.png)




