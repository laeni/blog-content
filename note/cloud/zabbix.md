# Zabbix Server

## 安装

### 初始化

1. 创建数据库和用户，并进行相关设置

   ```sh
   $ mysql -uroot -p<password>
   mysql> create database zabbix character set utf8mb4 collate utf8mb4_bin;
   mysql> create user 'zabbix'@'%' identified by '<password>';
   mysql> grant all privileges on zabbix.* to 'zabbix'@'%';
   mysql> SET GLOBAL log_bin_trust_function_creators = 1; # 必须设置，否则初始化时无法成功创建相关的表结构
   mysql> quit;
   ```

2. 初始化数据库

   ```sh
   # https://hub.docker.com/r/zabbix/zabbix-server-mysql
   $ docker run -d \
     --name zabbix-server \
     -e DB_SERVER_HOST="172.17.0.1" `# MySQL服务器的IP或DNS名称。默认值为“mysql-server”`\
     -e DB_SERVER_PORT="3306" `# MySQL服务器的端口。默认情况下，值为“3306”`\
     -e MYSQL_USER="zabbix" `# MySQL 用户名`\
     -e MYSQL_PASSWORD="zabbix" `# MySQL 密码`\
     -e MYSQL_DATABASE="zabbix" `# MySQL 数据库` \
     -p 10051:10051 \
     -e ZBX_WEBSERVICEURL="http://172.17.0.1:10053/report" \
     --init \
     -v /etc/localtime:/etc/localtime \
     zabbix/zabbix-server-mysql:6.0.27-ubuntu
   ```

3. 取消数据库的临时设置

   ```sh
   $ mysql -uroot -p<password>
   mysql> SET GLOBAL log_bin_trust_function_creators = 0;
   mysql> quit;
   ```

4. 重新启动

   去除`--init`参数。

# Zabbix Web

```sh
# https://hub.docker.com/r/zabbix/zabbix-web-nginx-mysql
$ docker run -d \
  --name zabbix-web \
  -e DB_SERVER_HOST="172.17.0.1" `# MySQL服务器的IP或DNS名称。默认值为“mysql-server”`\
  -e DB_SERVER_PORT="3306" `# MySQL服务器的端口。默认情况下，值为“3306”`\
  -e MYSQL_USER="zabbix" `# MySQL 用户名`\
  -e MYSQL_PASSWORD="zabbix" `# MySQL 密码`\
  -e MYSQL_DATABASE="zabbix" `# MySQL 数据库` \
  -e ZBX_SERVER_HOST="172.17.0.1" `# Zabbix server的IP或DNS名称。默认值为"zabbix-server"`\
  -e ZBX_SERVER_PORT="10051" `# Zabbix server监听的端口。默认值为"10051"`\
  -e PHP_TZ="Asia/Shanghai" `# PHP 格式的时区。默认值为“Europe/Riga”`\
  -P `# 10051`\
  `#-e ZBX_WEBSERVICEURL="http://zabbix-web-service:10053/report"` \
  --init \
  -v /etc/localtime:/etc/localtime \
  zabbix/zabbix-web-nginx-mysql:6.0.27-ubuntu
```

