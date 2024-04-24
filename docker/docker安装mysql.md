# docker安装mysql

## MySQL

* 搜索镜像

```shell
docker search mysql
```

* 拉取mysql镜像

``` shell
 docker pull mysql:x.x.x
```

* my.cnf
  ```conf
  [mysqld]
  sql_mode = "STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION"
  default-time-zone='+08:00'
  ```

* 创建容器并运行
  ```shell
  docker run -d --restart always -p 3306:3306 -v /data/docker/mysql/data:/var/lib/mysql -v /data/docker/mysql/conf:/etc/mysql/conf.d -v /data/docker/mysql/logs:/var/log/mysql -e MYSQL_ROOT_PASSWORD=123456 --name=mysql8.3.0 mysql:8.3.0
  ```
  
  > 命令行中指定密码的安全问题 
  > https://mysql.net.cn/doc/refman/5.7/en/docker-mysql-more-topics.html

* 修改密码
  ```shell
  update user set authentication_string=PASSWORD("xxx") where User='root'
  ```

  > -p 3306:3306：将主机的3306端口映射到docker容器的3306端口。
  >
  > --name mysql：运行服务名字
  >
  > -v /mydocker/mysql/conf:/etc/mysql/conf.d ：将 容器的 /etc/mysql/conf.d 目录挂载到主机 /mydocker/mysql/conf 目录
  >
  > -e MYSQL_ROOT_PASSWORD=123456：初始化 root 用户的密码。

* 进入容器
  ```shell
  # 进入容器内部
  docker exec -it mysql8.3.0 /bin/bash
  ```
  
* 操作数据库
  > 在容器内部操作

  ```shell
  # 连接到mysql
  mysql -u root -p
  # 创建数据库
  CREATE DATABASE mydatabase CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
  # 查看数据库列表
  show databases;
  # 使用数据库
  use mydatabase;
  # 导入sql
  source /path/something.sql;
  
  show tables;
  ```
