# docker安装redis

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
docker run -d --name myredis -p 6379:6379 -v /var/lib/docker/volumes/redis/data:/data -v /usr/local/docker/redis/conf/redis.conf:/etc/redis/redis.conf redis:xxx redis-server /etc/redis/redis.conf 
```

