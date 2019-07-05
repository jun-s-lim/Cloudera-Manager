# Prequel

1. yum update
```bash
sudo yum update -y
```
2. change the run level to multi-user text mode [(참고문서)](https://askubuntu.com/questions/900985/how-can-i-simply-change-into-a-text-mode-runlevel-under-systemd)
```bash
sudo systemctl isolate multi-user.target
sudo systemctl isolate runlevel3.target
```
3. disable SELinux [(참고문서)](https://www.lesstif.com/pages/viewpage.action?pageId=6979732)
```bash
sudo vi /etc/sysconfig/selinux # SELINUX=disabled 수정
```
4. disable firewall [(참고문서)](https://linuxize.com/post/how-to-stop-and-disable-firewalld-on-centos-7/)
```bash
sudo systemctl stop firewalld
sudo systemctl disable firewalld
```
5. check vm.swappiness and update permanently as necessary. [(참고문서)](https://askubuntu.com/questions/103915/how-do-i-configure-swappiness)
```bash
cat /proc/sys/vm/swappiness
sudo vi /etc/sysctl.conf # vm.swappiness=1 추가
```
6. disable transparent hugepage support permanently [(참고문서)](https://www.thegeekdiary.com/centos-rhel-7-how-to-disable-transparent-huge-pages-thp/)
```bash
cat /sys/kernel/mm/transparent_hugepage/enabled
sudo vi /etc/default/grub # transparent_hugepage=never 내용 추가
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
sudo reboot
cat /proc/cmdline # 
```
7. check to see that nscd service is running [(참고문서)](http://gurukaybee.blogspot.com/2017/05/rhel7-install-nscd-name-service-cache.html)
```bash
sudo yum install nscd -y
sudo systemctl start nscd
sudo systemctl enable nscd
sudo systemctl status nscd
```
8. check to see that ntp service is running (disable chrony as necessary) [(참고문서)](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/s1-disabling_chrony)
```bash
sudo systemctl stop chronyd
sudo systemctl disable chronyd
sudo yum install ntp -y
sudo systemctl start ntpd
sudo systemctl enable ntpd
```
9. disable IPV6 [(참고문서)](https://linuxhint.com/disable_ipv6_centos7/)
```bash
ip addr | grep inet6
sudo vi /etc/default/grub # ipv6.disable=1 내용 추가
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
# sudo reboot
ip addr | grep inet6 # 아무것도 안나오면 성공
```
10. setup a private/public key
```bash
# 서버들을 미리 호스트에 등록하자
# 각 서버 접속되게 키생성 및 연결
ssh-keygen -t rsa
# 엔터 계속

# 아래 두개 파일이 생성됨. 파일을 각서버에 넣음
.ssh/id_rsa
.ssh/id_rsa.pub

# .ssh/id_rsa.pub 내용을 .ssh/authorized_keys에 추가
chmod 600 .ssh/id*
```
11. update /etc/hosts
```bash
# 각 노드 /etc/hosts 등록
# ip [tab] FQDN [tab] shortcut
# ex)
# 172.31.x.x    h1.my.prac h1
# 172.31.x.x    h2.my.prac h2
# 172.31.x.x    h3.my.prac h3
# 172.31.x.x    h4.my.prac h4
# 172.31.x.x    h5.my.prac h5
```
12. change each hostname
```bash
# 각 노드 hostname 변경
sudo vi /etc/hostname
sudo reboot
```
13. add known hosts
```bash
# 각 노드에서 다른 노드 ssh 실행
ssh centos@h1
ssh centos@h2
ssh centos@h3
ssh centos@h4
ssh centos@h5
```

# Install

## Install JDK 1.8 (All nodes)

[참고문서](https://www.cloudera.com/documentation/enterprise/5-15-x/topics/cdh_ig_jdk_installation.html#topic_29_1)

1. install jdk
```bash
# Installing the JDK Manually
# jdk를 local 다운로드 받아서 각 노드에 복사
sudo mkdir -p /usr/java
sudo tar xvfz /home/centos/jdk-8u202-linux-x64.tar.gz -C /usr/java/
```
2. setup java path
```bash
# JAVA 경로 지정
sudo vi /etc/profile # export JAVA_HOME=/usr/java/jdk1.8.0_202 추가
source /etc/profile
env | grep JAVA_HOME
# JAVA_HOME 출력되면 성공
```

## Install Database

[참고문서](https://www.cloudera.com/documentation/enterprise/5-15-x/topics/install_cm_mariadb.html)

1. install mariadb server
```bash
# https://linuxize.com/post/install-mariadb-on-centos-7/
# mariadb install
sudo yum install mariadb-server
sudo systemctl stop mariadb

ll /var/lib/mysql/
sudo rm -f /var/lib/mysql/ib_logfile*
```
2. configura mariadb server
```bash
sudo vi /etc/my.cnf
```
```conf
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
```
3. start mariadb and setup mariadb secure
```bash
sudo systemctl start mariadb
sudo systemctl enable mariadb
sudo systemctl status mariadb
sudo mysql_secure_installation

# 순서대로 입력
# 1. enter
# 2. Y, root 패스워드
# 3. Y
# 4. N
# 5. Y
# 6. Y
```
4. install the MySQL JDBC Driver for MariaDB (All nodes)
```bash
# https://dev.mysql.com/downloads/connector/j/5.1.html에서 파일 다운로드 후 각 노드에 복사
sudo mkdir -p /usr/share/java/
tar xvf mysql-connector-java-5.1.47.tar.gz
sudo cp mysql-connector-java-5.1.47/mysql-connector-java-5.1.47.jar /usr/share/java/mysql-connector-java.jar
sudo ls /usr/share/java/
```
5. create Databases for Cloudera Software
```bash
mysql -u root -p
```
```sql
-- services
-- Cloudera Manager Server	scm	scm
-- Activity Monitor	amon	amon
-- Reports Manager	rman	rman
-- Hue	hue	hue
-- Hive Metastore Server	metastore	hive
-- Sentry Server	sentry	sentry
-- Cloudera Navigator Audit Server	nav	nav
-- Cloudera Navigator Metadata Server	navms	navms
-- Oozie	oozie	oozie

CREATE DATABASE scm DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
CREATE DATABASE amon DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
CREATE DATABASE rman DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
CREATE DATABASE hue DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
CREATE DATABASE metastore DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
CREATE DATABASE sentry DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
CREATE DATABASE nav DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
CREATE DATABASE navms DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
CREATE DATABASE oozie DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;

GRANT ALL ON scm.* TO 'scm'@'%' IDENTIFIED BY 'scm';
GRANT ALL ON amon.* TO 'amon'@'%' IDENTIFIED BY 'amon';
GRANT ALL ON rman.* TO 'rman'@'%' IDENTIFIED BY 'rman';
GRANT ALL ON hue.* TO 'hue'@'%' IDENTIFIED BY 'hue';
GRANT ALL ON metastore.* TO 'hive'@'%' IDENTIFIED BY 'hive';
GRANT ALL ON sentry.* TO 'sentry'@'%' IDENTIFIED BY 'sentry';
GRANT ALL ON nav.* TO 'nav'@'%' IDENTIFIED BY 'nav';
GRANT ALL ON navms.* TO 'navms'@'%' IDENTIFIED BY 'navms';
GRANT ALL ON oozie.* TO 'oozie'@'%' IDENTIFIED BY 'oozie';

SHOW DATABASES;
SHOW GRANTS FOR 'hive'@'%';
```

## Install CM softwares

[참고문서](https://www.cloudera.com/documentation/enterprise/5-15-x/topics/install_cm_server.html)

1. download repository
```bash
sudo wget https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/cloudera-manager.repo -P /etc/yum.repos.d/
vi /etc/yum.repos.d/cloudera-manager.repo
# baseurl 에 주소를 5.15.2 로 변경
# 실수로 변경 안한 상태로 yum 명령어를 실행했다면, 아래와 같이 캐시 삭제 필요
# rm -rf /var/cache/yum/*
# yum repolist
sudo rpm --import https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/RPN-GPG-KEY-cloudera
```
2. install CM master server
```bash
sudo yum install cloudera-manager-daemons cloudera-manager-server
```
3. install CM agent
```bash
sudo yum install cloudera-manager-daemons cloudera-manager-agent
```
4. configure the Cloudera Manager Agent to point to the Cloudera Manager Server
```bash
sudo vi /etc/cloudera-scm-agent/config.ini
# server_host, server_port: CM Server 노드 정보 입력
```
4. start CM agent
```bash
sudo systemctl start cloudera-scm-agent
```

## Start CM

[참고문서](https://www.cloudera.com/documentation/enterprise/5-15-x/topics/install_software_cm_wizard.html)

1. preparing the Cloudera Manager Server Database
```bash
sudo /usr/share/cmf/schema/scm_prepare_database.sh mysql scm scm
```
2. start Cloudera Manager Server
```bash
sudo systemctl start cloudera-scm-server
```
3. observe the startup process
```bash
sudo tail -f /var/log/cloudera-scm-server/cloudera-scm-server.log

# 아래 로그가 보인다면 성공
# INFO WebServerImpl:com.cloudera.server.cmf.WebServerImpl: Started Jetty server.
```
4. in a web browser, go to http://<server_host>:7180

# Cluster configuration using CM

## Wizard

1. 로그인: admin/admin
2. 패키지: impala 패키지 선택, **spark 패키지 제외 - 추후 커스텀 설정 후 설치 필요함**
3. 역할 할당 사용자 지정하기 [(참고문서)](https://www.cloudera.com/documentation/enterprise/5-15-x/topics/cm_ig_host_allocations.html#host_role_assignments)
4. 데이터베이스 설정 - 호스트, 데이터베이스, 유저, 패스워드 입력 - 테스트 연결: CM에서 각 서비스의 패키지 설치를 위한 설정

### Parcel 설치 방법

1. parcel로 이동 (우상단 선물 그림 아이콘)
2. 해당 패키지에서 Download 버튼 클릭
3. Distribute 버튼 클릭
4. Activate 버튼 클릭

## Setup Sqoop

1. Add Service로 이동
2. Sqoop1 패키지 선택 (Sqoop2는 deprecated 될 가능성 있음)
3. Gateway: 모든 노드

## Setup Impala

1. Add Service로 이동
2. 역할 선택 -> statestore: CM 노드, metastore: CM 노드, impalad: 모든 데이터 노드
3. Hue 서비스 > 구성으로 이동
4. Impala 서비스 none -> Impala 선택 후 변경 내용 저장
5. 서비스 재시작

## Setup Kafka

5. Add Service로 이동
6. Kafka Broker 설치: 모든 데이터 노드
7. 옵션 설정은 전부 default로 진행

## Setup Spark

*주의*: CM에서 spark 바로 설치하면 1.6버전이 설치됨. 2.x 설치하려면 아래를 진행해야한다.

* [참고문서](https://www.cloudera.com/documentation/spark2/latest/topics/spark2.html)
* [requirements](https://www.cloudera.com/documentation/spark2/latest/topics/spark2_requirements.html#requirements)

1. scala 2.11 이상 설치: 모든 노드
```bash
cd
wget http://downloads.lightbend.com/scala/2.11.8/scala-2.11.8.rpm
sudo yum install scala-2.11.8.rpm -y
```
2. parcels > configuration > remote parcel [(참고문서)](https://www.cloudera.com/documentation/spark2/latest/topics/spark2_packaging.html)
3. 파일 다운로드
```bash
cd /opt/cloudera/csd
wget http://archive.cloudera.com/spark2/csd/SPARK2_ON_YARN-2.4.0.cloudera2.jar
```
4. csd 파일 권한 변경 [(참고문서)](https://www.cloudera.com/documentation/spark2/latest/topics/spark2_installing.html)
```bash
chown cloudera-scm:cloudera-scm SPARK2_ON_YARN-2.4.0.cloudera2.jar
chmod 644 SPARK2_ON_YARN-2.4.0.cloudera2.jar
```
5. CM 서비스 재시작
```bash
sudo systemctl restart cloudera-scm-server
```
6. Add Service - 스파크
7. Spark2, HDFS, Hive, Zookeeper 선택
8. 진행 계속

# Test out Cluster

1. create user “training” in linux (All nodes)
```bash
sudo su
adduser training
usermod -aG wheel training
exit
sudo su training
groups
# training wheel -> wheel 추가 확인
```
2. create user “training” in hdfs
```bash
su hdfs
hdfs dfs -mkdir /user/training
hdfs dfs -chown training:training /user/training
hdfs dfs -ls /user
```
3. create the sample tables that will be used for the rest of the test
```bash
# zip 파일들을 미리 1번 노드에 업로드 함
sudo mv *.zip /training
sudo chmod training:training *.zip
su - training
unzip -Z authors.sql.zip
unzip -Z posts23.sql.zip
mysql -u root -p
```
```sql
CREATE DATABASE test DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
use test;
source /home/training/authors.sql
source /home/training/posts23.sql

create user 'training'@'localhost' identified by 'training';
create user 'training'@'%' identified by 'training';
grant all on test.* to 'training'@'%' identified by 'test';
select user,host from mysql.user where user='training' ;
exit;
```
4. extract tables posts from the database and create Hive tables (managed)
```bash
# sqooping posts table
sqoop import --connect jdbc:mysql://h1:3306/test \
--username root \
-P \
--split-by id \
--columns id,author_id,title,description,content,date \
--table posts \
--fields-terminated-by "\t" \
--hive-table default.posts \
--create-hive-table \
--hive-import
```
5. extract tables authors from the database and create Hive tables (external)
```bash
# sqooping authors table
sqoop import --connect jdbc:mysql://h1:3306/test \
--username root \
-P \
--split-by id \
--columns id,first_name,last_name,email,birthdate,added \
--table authors \
--fields-terminated-by "\t" \
--hive-table default.authors \
--create-hive-table \
--hive-import
```
```sql
ALTER TABLE default.authors SET TBLPROPERTIES('EXTERNAL'='TRUE');
```
```bash
# copy
hdfs dfs -cp /user/hive/warehouse/authors /user/training/authors
```
```sql
-- change hdfs path on schema
USE default;
DESC extended default.authors;
ALTER TABLE default.authors SET LOCATION "/user/training/authors";
```
6. test hive query
```sql
create table results
stored as textfile
as select a.id as id, a.first_name as fname, a.last_name as lname, count(*)
 from authors a
 join posts p on (a.id = p.author_id)
 group by a.id, a.first_name, a.last_name
 ;
```