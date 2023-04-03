# docker安装mysql、redis

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
docker run -d --name=mysqlx.x.x -p 3306:3306 -v mysqldata:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=password mysql:x.x.x
```

update user set authentication_string=PASSWORD("xxx") where User='root';

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



## Redis

 ```shell 
 docker pull redis
 ```

 

*  新建一个redis目录，用于存放redis.conf

```dos
mkdir /var/lib/redis
cd /var/lib/redis
```

```bash
#注释掉这部分，这是限制redis只能本地访问
# bind 127.0.0.1 

#默认yes，开启保护模式，限制为本地访问
protected-mode no 

#默认no，改为yes意为以守护进程方式启动，可后台运行，除非kill进程，改为yes会使配置文件方式启动redis失败
daemonize no

#数据库个数（可选）
databases 16 

#输入本地redis数据库存放文件夹（可选）
dir  ./ 

#redis持久化（可选）
appendonly yes 

logfile "access.log"

#redis持久化（设置成你自己的密码）
requirepass password
```

> redis-server --appendonly yes:在容器执行redis-server启动命令，并打开redis持久化配置



```bash
docker run --name myredis -p 6379:6379 -v /var/lib/docker/volumes/redis/data:/data -v /usr/local/docker/redis/conf/redis.conf:/etc/redis/redis.conf -d redis redis-server /etc/redis/redis.conf 
```

