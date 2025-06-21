

这里使用 mysql 提供数据库服务，下面是配置说明（公用服务器上的数据库已经上线了，可以直接使用，更偏爱本地的同学也可以按照下面，在本地构建一致的环境）

## Windows

直接在 mysql 官网下载即可，下载之后安装，运行 MySQL Configurator，使用默认配置，密码设置为 sysu-aicpm2025（和校内服务器密码一致）

![](../assets/KV7zbSNnRo4ygxx8gcfc9Ma7n3e.png)

然后进入 MySQL Command Line，创建数据库 sysu-aicpm2025

```
CREATE DATABASE `sysu-aicpm2025` DEFAULT CHARACTER SET utf8  COLLATE utf8_general_ci;
```

## docker

```bash
## 安装
sudo docker pull mysql:latest
sudo docker run -itd --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=xxxxxxxxx(在公开文档上密码不可见) mysql:latest
## 允许远程访问
sudo docker exec -it mysql bash 
echo "[mysqld] bind-address = 0.0.0.0" > /etc/mysql/conf.d/mysqld.cnf
mysql -h localhost -u root mysql -p
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;
exit
exit
sudo docker restart mysql
```

连接方式：

```bash
mysql -h 172.18.198.206 -u root mysql -p
```

创建数据库

```
CREATE DATABASE `sysu-aicpm2025` DEFAULT CHARACTER SET utf8  COLLATE utf8_general_ci;
```
