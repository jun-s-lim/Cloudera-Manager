# These are the commands for AWS CM install
1. Install JDK on all machines
```bash
scp -i ~/KeyPair/SKT.pem ~/Downloads/jdk-8u201-linux-x64.rpm centos@ad3:. &
// From each of the nodes
sudo -i rpm -ivh /home/centos/jdk-8u201-linux-x64.rpm
```
>OR
```bash
yum list java*jdk-devel
sudo yum install -y java-1.8.0-openjdk-devel.x86_64
#sudo yum install -y oracle-j2sdk1.8
```

2. Configure repository only CM
```bash
sudo yum install -y wget
sudo wget https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/cloudera-manager.repo \
-P /etc/yum.repos.d/
```
3. change the baseurl within cloudera-manager.repo to fit the version you want to install
```bash
baseurl=https://archive.cloudera.com/cm5/redhat/6/x86_64/cm/5.7.4/
for example: https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/5.14.4/

sudo rpm --import \
https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/RPM-GPG-KEY-cloudera
java -version
```
4. Install Cloudera Manager
```bash
sudo yum install -y cloudera-manager-daemons cloudera-manager-server
```
5. Install MariaDB only CM
```bash
sudo yum install -y mariadb-server

sudo systemctl enable mariadb
sudo systemctl start mariadb
sudo /usr/bin/mysql_secure_installation
```
6. Install mysql connector all node
```bash
scp -i ~/KeyPair/SEBC_HP.pem ~/Downloads/mysql-connector-java-5.1.47.tar.gz centos@acm:. 
OR
sudo wget https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.47.tar.gz

// From acm node
cd /home/centos
tar zxvf mysql-connector-java-5.1.47.tar.gz
sudo mkdir -p /usr/share/java/
cd mysql-connector-java-5.1.47
sudo cp mysql-connector-java-5.1.47-bin.jar /usr/share/java/mysql-connector-java.jar
```
7. Create the databases and users in MariaDB only CM
```bash
mysql -u root -p

CREATE DATABASE scm DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON scm.* TO 'scm-user'@'%' IDENTIFIED BY 'somepassword';

CREATE DATABASE amon DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON amon.* TO 'amon-user'@'%' IDENTIFIED BY 'somepassword';

CREATE DATABASE rman DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON rman.* TO 'rman-user'@'%' IDENTIFIED BY 'somepassword';

CREATE DATABASE hue DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON hue.* TO 'hue-user'@'%' IDENTIFIED BY 'somepassword';

CREATE DATABASE metastore DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON metastore.* TO 'metastore-user'@'%' IDENTIFIED BY 'somepassword';

CREATE DATABASE sentry DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON sentry.* TO 'sentry-user'@'%' IDENTIFIED BY 'somepassword';

CREATE DATABASE oozie DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON oozie.* TO 'oozie-user'@'%' IDENTIFIED BY 'somepassword';

FLUSH PRIVILEGES;
SHOW DATABASES;
EXIT;

** Setup the CM database only CM
sudo /usr/share/cmf/schema/scm_prepare_database.sh mysql scm scm-user somepassword
sudo rm /etc/cloudera-scm-server/db.mgmt.properties
sudo systemctl start cloudera-scm-server

GRANT ALL ON *.* TO 'training'@'%' IDENTIFIED BY 'training';
```
