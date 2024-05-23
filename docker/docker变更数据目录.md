# docker变更数据目录
> 参考: 
> https://www.cnblogs.com/chuyiwang/p/17577020.html
> https://segmentfault.com/a/1190000039426040
> https://forums.docker.com/t/solved-docker-service-fail-to-start-after-update/134444/3

## 更改目录
新建如下文件, 或在已有文件中写入:

*/etc/docker/daemon.json*
```json
{
  "log-driver":"json-file",
  "log-opts": {"max-size":"1024m", "max-file":"10"},
  "data-root": "/data/docker/lib/docker/" // 数据存储的目录
}
```
如果此时docker尚未启动, 直接启动即可.

如果此时docker正在运行, 需要将原有数据迁移到新目录下.

```shell
# 1.停止docker服务
$ sudo systemctl stop docker

# 2.迁移旧数据到新目录下
# 如果使用cp, 需要复制文件权限和属性
$ sudo mv /var/lib/docker/ /data/docker/lib/
# $ sudo cp -arv /var/lib/docker/ /data/docker/lib/

# 3.启动服务
$ sudo systemctl start docker

```