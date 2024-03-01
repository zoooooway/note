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

* 创建数据卷：

  ```shell  
  docker volume create mysqldata
  ```

  * 查看数据卷列表:

  ```shell
  docker volume ls
  ```

  * 获取数据卷存储位置：

  ```shell
  docker inspect mysqldata
  ```

* 创建容器并运行

```shell
docker run -d --name=mysql8.3.0 -p 3306:3306 -v /data/docker/mysql/data:/var/lib/mysql -v /data/docker/mysql/conf:/etc/mysql/conf.d -v /data/docker/mysql/logs:/var/log/mysql -e MYSQL_ROOT_PASSWORD=ceRS2FsVkeg9 mysql:8.3.0
```

`update user set authentication_string=PASSWORD("xxx") where User='root'`;

> -p 3306:3306：将主机的3306端口映射到docker容器的3306端口。
>
> --name mysql：运行服务名字
>
> -v /mydocker/mysql/conf:/etc/mysql/conf.d ：将主机/mydocker/mysql录下的conf/my.cnf 挂载到容器的 /etc/mysql/conf.d
>
> -v /mydocker/mysql/logs:/var/log/mysql：将主机/mydocker/mysql目录下的 logs 目录挂载到容器的 /logs。
>
> -v /mydocker/mysql/data:/var/lib/mysql ：将主机/mydocker/mysql目录下的data目录挂载到容器的 /var/lib/mysql
>
> -e MYSQL_ROOT_PASSWORD=123456：初始化 root 用户的密码。
>
> -d mysql:5.7 : 后台程序运行mysql5.7
>
> --character-set-server=utf8mb4 ：设置字符集
>
> --collation-server=utf8mb4_unicode_ci：设置校对集

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
