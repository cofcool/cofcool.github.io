# Monitor

## Nagios

## Zabbix

https://www.zabbix.com/download?zabbix=4.0&os_distribution=ubuntu&os_version=16.04_xenial&db=mysql&ws=apache

Install and configure Zabbix server
a. Install Repository with MySQL database
documentation
# wget http://repo.zabbix.com/zabbix/3.4/ubuntu/pool/main/z/zabbix-release/zabbix-release_3.4-1+xenial_all.deb
# dpkg -i zabbix-release_3.4-1+xenial_all.deb
# apt update
b. Install Zabbix server, frontend, agent
# apt install zabbix-server-mysql zabbix-frontend-php zabbix-agent
c. Create initial database
documentation
# mysql -uroot -p
password
mysql> create database zabbix character set utf8 collate utf8_bin;
mysql> grant all privileges on zabbix.* to zabbix@localhost identified by 'password';
mysql> quit;

Import initial schema and data. You will be prompted to enter your newly created password.
# zcat /usr/share/doc/zabbix-server-mysql*/create.sql.gz | mysql -uzabbix -p zabbix
d. Configure the database for Zabbix server
DBPassword=password
e. Start Zabbix server and agent processes

Start Zabbix server and agent processes and make it start at system boot:
# systemctl restart zabbix-server zabbix-agent apache2
# systemctl enable zabbix-server zabbix-agent apache2
f. Configure PHP for Zabbix frontend
Edit file /etc/zabbix/apache.conf, uncomment and set the right timezone for you.
# php_value date.timezone Europe/Riga


Configure Zabbix frontend

Connect to your newly installed Zabbix frontend: http://server_ip_or_name/zabbix
Follow steps described in Zabbix documentation: Installing frontend


若无数据可能是在设置中未启用，或是agent配置错误：HostName，Ip端口

----

2020-03-24

CentOS 7

install:

rpm -Uvh https://repo.zabbix.com/zabbix/4.0/rhel/7/x86_64/zabbix-release-4.0-2.el7.noarch.rpm

yum install zabbix-server-mysql zabbix-web-mysql zabbix-agent 

mysql -uroot -p
create database zabbix character set utf8 collate utf8_bin;
grant all privileges on zabbix.* to zabbix@localhost identified by 'password';

zcat /usr/share/doc/zabbix-server-mysql*/create.sql.gz | mysql -uzabbix -p zabbix 


Configure the database for Zabbix server

Edit file /etc/zabbix/zabbix_server.conf
DBPassword=password 

Configure PHP for Zabbix frontend

Edit file /etc/httpd/conf.d/zabbix.conf, uncomment and set the right timezone for you.
# php_value date.timezone Europe/Riga 
php_value date.timezone Asia/Shanghai

systemctl restart zabbix-server zabbix-agent httpd
systemctl enable zabbix-server zabbix-agent httpd 

----

Ubuntu 16.04 

wget https://repo.zabbix.com/zabbix/4.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_4.0-3+xenial_all.deb
sudo dpkg -i zabbix-release_4.0-3+xenial_all.deb
sudo apt update 

sudo apt install zabbix-server-mysql zabbix-frontend-php zabbix-agent

Configure PHP for Zabbix frontend

Edit file /etc/zabbix/apache.conf, uncomment and set the right timezone for you.
# php_value date.timezone Europe/Riga 

# systemctl restart zabbix-server zabbix-agent apache2
# systemctl enable zabbix-server zabbix-agent apache2 

默认账号: Admin 密码: zabbix

### agent

https://www.zabbix.com/integrations/mysql

配置模版

https://git.zabbix.com/projects/ZBX/repos/zabbix/browse/templates

https://git.zabbix.com/projects/ZBX/repos/zabbix/browse/templates/db/mysql_agent

配置mysql

CREATE USER 'zbx_monitor'@'%' IDENTIFIED BY '<password>';
GRANT USAGE,REPLICATION CLIENT,PROCESS,SHOW DATABASES,SHOW VIEW ON *.* TO 'zbx_monitor'@'%';

sudo vim /etc/zabbix/zabbix_agentd.d/.my.cnf

[client]
user=zbx_monitor
password=<password>

sudo vim /etc/zabbix/zabbix_agentd.d/mysql_template.conf
# $1,$2 等值替换为真实值
UserParameter=mysql.ping[*], mysqladmin -h"$1" -P"$2" ping
UserParameter=mysql.get_status_variables[*], mysql -h"$1" -P"$2" -sNX -e "show global status"
UserParameter=mysql.version[*], mysqladmin -s -h"$1" -P"$2" version
UserParameter=mysql.db.discovery[*], mysql -h"$1" -P"$2" -sN -e "show databases"
UserParameter=mysql.dbsize[*], mysql -h"$1" -P"$2" -sN -e "SELECT COALESCE(SUM(DATA_LENGTH + INDEX_LENGTH),0) FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_SCHEMA='$3'"
UserParameter=mysql.replication.discovery[*], mysql -h"$1" -P"$2" -sNX -e "show slave status"
UserParameter=mysql.slave_status[*], mysql -h"$1" -P"$2" -sNX -e "show slave status"

Tomcat

sudo apt install zabbix-java-gateway

开启 JMX

Tomcat 开启 JMX 
CATALINA_OPTS="-Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=9000 -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false"