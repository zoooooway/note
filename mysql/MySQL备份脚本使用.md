# 数据库自动备份脚本使用

## 备份

### Linux

> 请先在测试环境进行测试

* **编写备份脚本**

根据情况调整以下内容中的参数，并保存为`backup.sh`并放置在数据库所在服务器上（位置无限制）

``` shell
echo '##########################################'
echo '###### The database is automatically backed up at 03:00 am every day ######'
echo '##########################################'
# Setting environment variables
db_user="username"
db_password="password"
# 数据库名称
db_name="test"
# docker容器名 未使用docker则删除此变量
db_container_name="mysql"
# 备份文件的存放位置
backup_dir="/usr/local/db/backup"
time="$(date +"%Y%m%d")"
echo 'Get system date: ' $time
if [ ! -e "$backup_dir" ]; then
mkdir $backup_dir
fi
cd $backup_dir
echo ${backup_dir}
echo 'backup started...' $(date "+%Y-%m-%d %H:%M:%S")
# docker安装mysql使用下列命令
docker exec ${db_container_name} mysqldump -u${db_user} -p${db_password} ${db_name} --quick --single-transaction > ${db_name}_${time}.sql
# 直接安装mysql使用下列命令
# mysqldump所在位置 -u${db_user} -p${db_password} ${db_name} --quick --single-transaction > ${db_name}_${time}.sql
find ${backup_dir} -name ${db_name}"*.sql" -type f -mtime +7 -exec rm -rf {} \; > /dev/null 2>&1
echo 'backup completed!' $(date "+%Y-%m-%d %H:%M:%S")
```

> P.S. 执行失败时请检查Windows与Linux符号差异问题，并且避免以文本文档（.txt）来新建脚本

* **授予脚本可执行权限**

进入脚本所在目录并执行以下命令：

``` shell
chmod u+x backup.sh
```

* **测试脚本**

执行命令：

``` shell
./backup.sh
```

观察控制台输出，并检查生成的SQL文件。

* **创建定时任务**

执行以下命令：

``` shell
crontab -e
```

在文本中插入以下内容：

``` tex
0 3 * * * /usr/local/db/backup.sh >> /usr/local/db/log/backup.log 2>&1
```

> 每天凌晨三点执行备份
>
> `> /usr/local/db/log/backup.log 2>&1`是将任务执行的日志输出到指定文件。如果不需要日志则可以去除这一段。

保存并退出，任务会自动运行。



***

## 导入备份数据

按普通SQL文件导入方式使用即可

**注意：**

* 调整SQL文件

有时，备份的SQL文件在开头会附带这一行：*mysqldump: [Warning] Using a password on the command line interface can be insecure.* 。如果有，则需要删除该行再执行导入操作。

* 出现 `Got a packet bigger than 'max_allowed_packet' bytes` 错误

打开数据库，执行以下查询：

``` sql
SELECT @@max_allowed_packet / 1024 / 1024;
```

得到当前最大允许读取包大小，如果小于你要导入的SQL文件大小，则执行以下命令来提高该限制大小：

``` sql
SET GLOBAL max_allowed_packet=1000000000;
```

> P.S. 该设置在数据库重启后失效

