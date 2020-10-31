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

![image](https://user-images.githubusercontent.com/30167661/97786777-38444a80-1bf1-11eb-8ccc-ade8544e034f.png)

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

![image](https://user-images.githubusercontent.com/30167661/97786852-b30d6580-1bf1-11eb-8838-02f34dc65678.png)


### c. Install Cloudera Manager


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


