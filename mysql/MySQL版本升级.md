# MySQL版本升级
简记一次 MySQL 的版本升级过程.
> 所有流程均建立在已备份的前提下. 如果可以, 请先在测试环境中试一遍流程.

## MySQL 使用压缩包安装

### 版本
MySQL 版本采用社区版, 当前版本: 8.0.20, 升级版本: 8.0.32. 
> 下载地址: https://downloads.mysql.com/archives/community/

### 流程
由于升级后版本是大于8.0.16的, 在这之后的 MySQL 版本对升级做了优化, 使得升级十分简单. 简单来说就是安装新版本后, MySQL Server 会自动做一系列的升级操作.

1. 首先, 备份所有需要备份的东西. 应该包括:
   * 数据库. 可以使用 MySQL 自带的导出工具来备份, 首先进入到 MySQL 安装目录下的 bin 目录, 然后执行:
    `./mysqldump -u${db_user} -p${db_password} ${db_name} > ${filename}.sql`
   * 配置文件. 为防止配置文件在操作时被误删, 也应该备份一下.
   * 原版本的文件. 如果占用空间不大,建议将原版本的目录直接全部备份. (以防万一, 备份做得越好, 操作时越安心)

2. 下载新版本的压缩包, 地址见上方下载地址.
3. 将 MySQL 进程停止. 这里网上有很多教程, 大部分是使用 `mysqld stop` 来停止, 但我这边会报错, 主要是当时安装的时候的配置没弄好. 于是我简单的直接把 MySQL 进程 `kill` 了(不推荐, 虽然我没遇到问题, 但不能保证你不会出现问题).
4. 将下载好的 MySQL 压缩包解压, 并且复制到当前 MySQL 安装目录下去覆盖原先的文件. 
   ```bash
   > tar -xf mysql-8.0.32-linux-glibc2.12-x86_64.tar.xz
   > mv mysql-8.0.32-linux-glibc2.12-x86_64/* /{MySQL安装目录}
   ```
   如果 `mv` 命令报错, 提示原目录下某个文件夹不为空, 那么就把那个文件删除或者移到别处去备份.

5. 进入到 MySQL 安装目录下的 bin 目录, 执行 `./mysqld_safe --defaults-file=配置文件 --user=mysql &`
6. 查看 MySQL 的输出日志, 日志位置取决于你的配置文件. 如果一切正常, 那么你会看到这样的输出:
   ```log
   2023-04-18T08:39:23.225573Z 4 [System] [MY-013381] [Server] Server upgrade from '80020' to '80032' started.
   2023-04-18T08:39:29.973308Z 4 [System] [MY-013381] [Server] Server upgrade from '80020' to '80032' completed.
   ......
   2023-04-18T08:39:30.129941Z 0 [System] [MY-010931] [Server] /usr/mysql/mysqld: ready for connections. Version: '8.0.32'  socket: '/tmp/mysql.sock'  port: 3306  MySQL Community Server - GPL.
   ```

7. 最后, 进入到 MySQL 安装目录下的 bin 目录. 执行 `./mysql -uroot -p`, 连接到 MySQL, 查看 MySQL 的版本.
   

## MySQL 使用 Docker 安装
MySQL 版本采用社区版, 当前版本: 8.0.27, 升级版本: 8.0.33. 
> 镜像地址: https://hub.docker.com/_/mysql/tags

我也尝试过5.7版本之间的小版本升级, 流程是一样的.

### 流程
采用 Docker 安装相对于使用压缩包安装来说, 在升级方面上方便了不少. 因为我们可以保留原来的容器, 这样失败时直接重启原来的容器就好了. 并且一般我们都会将数据挂载到数据卷或者宿主机上, 因此, 数据备份也十分简单.

* 首先先停止原来的 MySQL 容器, 容器保留以防止升级失败. 
* 接下来先创建一个数据卷用于拷贝原来容器中的数据(当然你也可以不使用数据卷的方式来挂载, 可以在容器启动时使用 -v 命令来进行容器和宿主机直接的映射, 原理是一样的). 执行以下命令来创建数据卷: 
  ```shell
  docker volume create mysql_data_new 
  ```
* 将原来容器中的数据直接复制到新数据卷中. 如果原来的容器是通过数据卷进行挂载的, 可以使用执行以下命令查看容器信息来查看挂载的数据卷:
  ```shell
  docker inspect 原来的容器名
  ```
  在容器信息中, 你可能会发现类似这样一段信息:
  ```json
  "Mounts": [
            {
                "Type": "volume",
                "Name": "mysql_data",
                "Source": "/var/lib/docker/volumes/mysql_data/_data",
                "Destination": "/var/lib/mysql",
                "Driver": "local",
                "Mode": "z",
                "RW": true,
                "Propagation": ""
            }
        ],
  ```
  这个 "Source" 的值就是数据卷在宿主机上的位置, 我们将其拷贝到新的数据卷中:
  ```shell
  cp -r /var/lib/docker/volumes/mysql_data/ /var/lib/docker/volumes/mysql_data_new/
  ```
  如果使用的是 -v 命令来进行容器和宿主机直接的映射, 也是一样的道理, 找到挂载的目录复制过去就好了. 
  
  那假如我之前运行容器时没有挂载数据卷和目录映射呢? 老实说这很少见, 但如果真是这样的情况, 你可以将原来的容器重新启动, 利用 `docker cp` 命令将容器内部的数据拷贝到宿主机上. 但需要注意一点, 最好确保你这样操作的时候外部没有在对 MySQL 进行操作了, 否则拷贝的数据可能出错导致后续升级时启动新容器失败.
* 将要安装的新版本 MySQL 镜像拉取下来, 启动容器:
  ```shell
  docker run -d --name=mysql8.0.33 -p 3306:3306 -v mysql_data_new:/var/lib/mysql  mysql:8.0.33
  ```

如果一切顺利, 那么这样就完成升级了, 很简单对吧 ;).

### 问题
假如没那么顺利, 那么你可能会在查看容器启动日志时发现一些报错: 

#### 原来 MySQL 中的数据在新版本 MySQL 中出错
```log
MYSQL: Upgrade is not supported after a crash or shutdown with innodb_fast_shutdown = 2
```

解决办法: 进入 MySQL 挂载的数据卷目录, 删除 ib_logfile 开头的文件(通常情况下是: ib_logfile0 和 ib_logfile1 这两个文件)
> 参考链接: https://serverfault.com/a/1110090/1033998

删除完这两个文件你可能又会碰到一个新的问题:
```log
Neither found #innodb_redo subdirectory, nor ib_logfile* files in ./
```
参照此链接解决: https://askubuntu.com/a/1430285/1652011

#### 原来 MySQL 中的数据在新版本 MySQL 中出错
```log
Your database may be corrupt or you may have copied the InnoDB tablespace but not the InnoDB redo log files. 
Please refer to http://dev.mysql.com/doc/refman/8.0/en/forcing-innodb-recovery.html for information about forcing recovery.
```

或者这样:
```log
Tablespace 'xxx/xxx' Page [page id: space=261, page number=1196] log sequence number 14243863172 is in the future! Current system log sequence number 340263626.
```

解决办法: 在 MySQL 的配置文件中添加:
```
[mysqld]
innodb_force_recovery=1
```
> 参考链接: https://dba.stackexchange.com/a/182800

添加完成后, 重新启动容器. 再次查看容器日志, 如果容器输出提示你将`innodb_force_recovery`的值改为 0 的信息. 在 MySQL 的配置文件中添加
```
[mysqld]
innodb_force_recovery=0
```
再次重启容器.