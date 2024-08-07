# 常用linux命令合集
## 查看服务器信息
### 版本名称和版本号
```shell
lsb_release -a
```



## 查看文件和文件夹大小
有时想查看指定目录到底占用了多少磁盘，可以使用如下命令：
```shell
du -sh *
```
`du`显示每个文件和目录的磁盘使用空间

一些额外命令：

* 文件从大到小排序:

```shell
du -sh * |sort -rh
``` 

* 显示总和的大小且易读:

```shell
du -sh .
``` 

> `du`命令也是查看使用空间的，但是与`df`命令不同的是Linux `du`命令是对文件和目录磁盘使用的空间的查看，还是和df命令有一些区别的。

## 查看一个进程是否存在
查看redis进程是否存在：
```shell
ps -ef | grep redis | grep -v grep | awk '{print $2}'
```
> 上述命令利用管道符将输入与输出连接。
> `$0`这个变量包含执行过程中当前行的文本内容。`$n`当前记录的第n个字段，比如n为1表示第一个字段，n为2表示第二个字段。
> 例如: 
> `$0`输出 `systemd+ 3010504 3010485  0 Aug13 ?        02:38:51 redis-server *:6379` 
> `$1`输出 `systemd+`
### grep 
常用的文本搜索工具 
* 输出除match_pattern以外的所有行
  ```shell 
  grep -v "match_pattern" file_name
  ```

### awk
文本和数据进行处理的编程语言，用于在linux/unix下对文本和数据进行处理。
* 打印每一行的第二和第三个字段
  ```shell 
  awk '{ print $2,$3 }' filename
  ```


## 删除过期文件 
```shell
find 文件所在路径 -type f -name *.jpg -mtime +210  -print -delete
```
> 最后一次修改时间: -mtime (天)  -mmin (分钟)
> 最近一次访问时间: -atime (天)  -amin (分钟)

## 流量监控
可以使用 **iftop**.

* 安装: `yum install iftop`.
* 执行 `ifconfig` 查看网卡信息.
* ` iftop -i 网卡名称 -P`.


## 通过系统启动信息来排查问题
```shell
# 显示java相关的开机过程信息
dmesg -T | grep java
```
> -T 显示时间