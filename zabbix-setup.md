## Step 1: Install Zabbix server, frontend, and agent
`Install Zabbix .deb package on your Ubuntu OS (24.04, 22.04, 20.04, 18.04 and 16.04 are supported).`

```bash
Zabbix 7.0 LTS version (supported until 	June 31, 2029)
wget https://repo.zabbix.com/zabbix/7.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_7.0-1+ubuntu$(lsb_release -rs)_all.deb
sudo dpkg -i zabbix-release_7.0-1+ubuntu$(lsb_release -rs)_all.deb
sudo apt update
sudo apt -y install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent
```

## Step 2: Configure database

### a. Add the MySQL APT Repository and Install MySQL
```bash
wget https://dev.mysql.com/get/mysql-apt-config_0.8.30-1_all.deb
sudo dpkg -i mysql-apt-config_0.8.30-1_all.deb
sudo apt update
sudo apt install mysql-server -y
```
### b. Start MySQL Service

```bash
sudo systemctl start mysql
sudo systemctl enable mysql
```

### c. Secure MySQL Installation

```bash
sudo mysql_secure_installation
```

```
Set up the VALIDATE PASSWORD PLUGIN: Choose Y and select a level of password validation, or choose N if password is already set during installation or use Unix socket authentication
Set root password: Choose Y and enter a strong password for the root user.
Remove anonymous users: Choose Y.
Disallow root login remotely: Choose Y.
Remove test database: Choose Y.
Reload privilege tables: Choose Y.
```

## Step 3: Verify MySQL Installation
```bash
mysql -uroot -p
password
mysql> create database zabbix character set utf8mb4 collate utf8mb4_bin;
mysql> create user zabbix@localhost identified by 'password';
mysql> grant all privileges on zabbix.* to zabbix@localhost;
mysql> flush privileges
mysql> set global log_bin_trust_function_creators = 1;
mysql> quit;
```

##  import initial schema and data.
```bash
zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -uzabbix -p zabbix
```
## Disable log_bin_trust_function_creators option after importing database schema.
```bash
mysql -uroot -p
password
mysql> set global log_bin_trust_function_creators = 0;
mysql> quit;
```

## Configure the database for Zabbix server
```
vim /etc/zabbix/zabbix_server.conf
DBPassword=password
```
## Start Zabbix server and agent processes
### Start Zabbix server and agent processes and make it start at system boot.
```bash
systemctl restart zabbix-server zabbix-agent apache2
systemctl enable zabbix-server zabbix-agent apache2
```
