# MySQL8版本升级
简记一次 MySQL 的版本升级过程.
> 所有流程均建立在已备份的前提下. 如果可以, 请先在测试环境中试一遍流程.

## 版本
MySQL 版本采用社区版, 当前版本: 8.0.20, 升级版本: 8.0.32. 
> 下载地址: https://downloads.mysql.com/archives/community/

安装方式是使用压缩包安装. 

## 流程
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
   