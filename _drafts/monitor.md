# Monitor

## Nagios

## Zabbix

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
