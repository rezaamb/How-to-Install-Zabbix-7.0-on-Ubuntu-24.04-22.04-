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
### or do this command instead of secure installation :
```bash
mysql -u root -p
ALTER USER 'root'@'localhost' IDENTIFIED WITH caching_sha2_password BY 'Far@1404!2025';
FLUSH PRIVILEGES;
exit;
```
these 2 commands are for PREMISSIONS :
```bash
sudo chown -R mysql:mysql /var/lib/mysql
sudo chmod 750 /var/lib/mysql
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

##  Step 4: import initial schema and data.
```bash
zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -uzabbix -p zabbix
```
## Step 5: Disable log_bin_trust_function_creators option after importing database schema.
```bash
mysql -uroot -p
password
mysql> set global log_bin_trust_function_creators = 0;
mysql> quit;
```

## Step 6: Configure the database for Zabbix server
```
vim /etc/zabbix/zabbix_server.conf
DBPassword=password
```


## Step 7: Configure firewall
If you have a UFW firewall installed on Ubuntu, use these commands to open TCP ports: 10050 (agent), 10051 (server), and 80 (frontend):
```bash
apt install ufw -y
```
```bash
ufw allow 10050/tcp
ufw allow 10051/tcp
ufw allow 80/tcp
ufw reload
```

###  Step 8: Start Zabbix server and agent processes and make it start at system boot.
```bash
systemctl restart zabbix-server zabbix-agent apache2
systemctl enable zabbix-server zabbix-agent apache2
```

##  Step 9: Connect to your newly installed Zabbix frontend using URL ```“http://server_ip_or_dns_name/zabbix”```
![image](https://github.com/user-attachments/assets/db09874d-9894-4e6d-a668-12a8fd5277ee)

## Step 10: if you get the error “zabbix Locale for language “en_US” is not found on the web server. Tried to set: en_US, en_US.utf8, en_US.UTF-8, en_US.iso885915, en_US.ISO8859-1” then use this command to fix it:
```
apt-get install -y locales && echo 'en_US.UTF-8 UTF-8' >> /etc/locale.gen && locale-gen && service apache2 restart
```
![image](https://github.com/user-attachments/assets/c258cb2b-9117-4f78-b759-7b191dfc591c)
![image](https://github.com/user-attachments/assets/8154e14f-076b-411a-ae08-0ce5d3af98bf)
![image](https://github.com/user-attachments/assets/db11d0cb-9888-4f0b-aa7f-975bdf60d248)
![image](https://github.com/user-attachments/assets/0678d469-2483-491a-ae7c-28d6b944a31d)
![image](https://github.com/user-attachments/assets/3226b474-892a-41cf-8c8f-952695e1c1c8)


## Step 11: Login to frontend using Zabbix default login credentials
Use Zabbix default admin username “Admin” and password “zabbix” (without quotes) to login to Zabbix frontend at URL “http://server_ip_or_dns_name/zabbix” via your browser.

![image](https://github.com/user-attachments/assets/fd42a7dc-0544-40aa-a15f-f6bf22340c2e)

and then you can change the PASSWORD in UI : Users Tab > Users > Change Password

## Step 12: Optimizing Zabbix Server (optional)

Don’t bother with this optimization if you are monitoring a small number of devices, but if you are planning to monitor a large number of devices then continue with this step.

Open “zabbix_server.conf” file with command: ```“sudo vim /etc/zabbix/zabbix_server.conf”``` and add this configuration anywhere in file:
```
StartPollers=100
StartPollersUnreachable=50
StartPingers=50
StartTrappers=10
StartDiscoverers=10
StartHTTPPollers=10
CacheSize=128M
HistoryCacheSize=64M
HistoryIndexCacheSize=32M
TrendCacheSize=32M
ValueCacheSize=256M
```


##  Step 13: Update Zabbix server (or Proxy) configuration file
Note: In case of Zabbix proxy, follow similar steps: edit ```zabbix_proxy.conf``` and restart the ```zabbix-proxy``` service

You need to configure Zabbix server for VMware monitoring. Open zabbix_server.conf file with command:
```“vim /etc/zabbix/zabbix_server.conf”``` and add these VMware parameters anywhere in the file:
```
StartVMwareCollectors=3
VMwareFrequency=60
VMwarePerfFrequency=60
VMwareCacheSize=32M
VMwareTimeout=120
```

##  Step 14:How to set vmware user password and url on zabbix 
![how_to_set_vmware_api_user_password_and_url_on_zabbix](https://github.com/user-attachments/assets/923dbb9a-3809-40aa-a1d2-ebc69f925f13)


