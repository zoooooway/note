# 使用docker安装MongoDB

## 版本
* MongoDB: 5.0.18
* Linux: 3.10.0-957.21.3.el7.x86_64 GNU/Linux
* Docker: 18.03.1-ce
  
为什么要选择`5.0.18`版本呢? 因为高于这个版本的我运行容器都会报错无法启动。比如下面这种报错信息:
```json
{
    "t": {
        "$date": "2023-06-14T12:16:35.314+00:00"
    },
    "s": "F",
    "c": "CONTROL",
    "id": 6384300,
    "ctx": "main",
    "msg": "Writing fatal message",
    "attr": {
        "message": "terminate() called. An exception is active; attempting to gather more information\n"
    }
}
{
    "t": {
        "$date": "2023-06-14T12:16:35.314+00:00"
    },
    "s": "F",
    "c": "CONTROL",
    "id": 6384300,
    "ctx": "main",
    "msg": "Writing fatal message",
    "attr": {
        "message": "std::exception::what(): Operation not permitted\nActual exception type: std::system_error\n\n"
    }
}
```

或者这种:
```json
{
    "t": {
        "$date": "2023-06-14T10:26:16.033+00:00"
    },
    "s": "I",
    "c": "CONTROL",
    "id": 31445,
    "ctx": "main",
    "msg": "Frame",
    "attr": {
        "frame": {
            "a": "55F5EB37D369",
            "b": "55F5E945B000",
            "o": "1F22369",
            "s": "_ZSt20__throw_system_errori.cold.24",
            "C": "std::__throw_system_error(int) [clone .cold.24]",
            "s+": "2F"
        }
    }
}
```

根据这些异常信息也没办法排查是哪里的问题，后来有帖子说是 Docker 的版本问题，升级版本就能解决。但我并不想升级 Docker，于是选择选择低版本的 MongoDB。

## 配置文件
配置文件参考[官网: Configuration File](https://www.mongodb.com/docs/manual/reference/configuration-options/#configuration-file)

参考配置:
```yaml
systemLog:
   destination: file
   path: "/var/mongodb/log/mongod.log"
   logAppend: true
storage:
   dbPath: "/var/mongodb/data/db"
   journal:
      enabled: true
net:
   bindIp: 127.0.0.1
   port: 27017
setParameter:
   enableLocalhostAuthBypass: false
```
说明: 
* net.bindIp 代表绑定的连接 ip 地址, 设置为 127.0.0.1 表示只允许本地连接, 如果需要远程访问, 则需要设置为远程主机的 ip 或者去掉这个设置
  > **警告**: 假如你正在公网服务器上进行进行操作, 在没有开启访问控制时(后面会提到), 建议设置只允许本地连接或者开启防火墙来限制远程访问. 否则你的 MongoDB 相当于是完全暴露在公网上的, 极有可能会被入侵.

* storage.dbPath 是 MongoDB 持久化数据的目录, systemLog.path 则是日志文件

## 构建
选择合适的镜像. 如果对版本没有要求, 直接执行 `docker pull mongo` 即可拉取最新的镜像. 
但如果你像我一样, Docker 的版本无法兼容最新版本的 MongoDB, 可以先去选择能安装版本的镜像(当然, 也可以选择升级 Docker 的版本), 所有的版本都可以在[官方仓库](https://hub.docker.com/_/mongo/tags)找到.
> 怎么知道能不能安装呢? 当然是拉取下来跑一下试试了, 不行就取下一个版本

拉取了镜像后, 在运行容器前, 需要先做点准备工作.

以我上面的配置为例, 我配置的日志文件和数据目录, 在运行容器前, 需要先创建这两个配置对应的文件夹和文件, 以避免在容器内部找不到对应目录或文件. 

同理, 还需要给容器能访问这些目录或者文件的权限.
可以执行 `chmod 文件 777` 来简单的给所有用户访问此文件的权限. 当然, 这有一定的安全隐患.

最后, 将配置文件放在将会挂载到容器里的目录下, 这样容器启动时就能读取这个文件来进行启动.

做完这些后, 就可以启动容器了. 执行 
```shell
docker run -d -m 3g --cpus="2" --restart=always --name mongodb -p 27317:27017 -v /var/mongodb:/var/mongodb mongo:5.0.18 --config /var/mongodb/mongod.conf
```

说明: 
* -m 和 --cpus 分别指定容器使用的最大内存和cpus核心数, 可以不加
* --restart=always 表示容器如果异常关闭会自动重启, 可以不加
* -p 27317:27017 表示将容器的27017端口映射到宿主机————也就是服务器的27317端口
  > 这么做出于安全性考虑, 因为很多网络攻击是通过特定的端口扫描来进行的, 像 MongoDB 的端口属于重点攻击对象, 因此换个端口能减少些风险(聊胜于无).
* -v /var/mongodb:/var/mongodb 表示将容器的 /var/mongodb 目录挂载到宿主机上的 /var/mongodb 目录下, 这个目录是根据配置文件的 dbPath 来定的, 挂载之后，即使删除了容器, 数据也还是在宿主机上的.
* --config 表示通过指定的配置文件启动 MongoDB, 需要注意的是, 这个路径是容器里的文件路径. 在前面的步骤中, 我们已经把配置文件放在挂载到容器内部目录下了, 因此容器能读到配置文件.

执行命令后, 可以观察配置文件中设置的 log 文件内容或者执行 `docker logs -f mongodb` 观察容器输出来确定容器是否正确启动.

## 开启访问限制
MongoDB 默认是没有开启身份验证的, 因此不少人的 MongoDB 都被非法入侵过(包括我). 
> 提起非法入侵, 在学习这种中间件或者服务的时候, 去 google 一下 xxx 攻击或者 xxx 入侵能看到不少相关例子, 吸取这些经验的话或许能避免后面栽跟头

开启身份验证可以参考官网, 写的非常详细: https://www.mongodb.com/docs/manual/tutorial/enable-authentication/

我使用的是 SCRAM (Salted Challenge Response Authentication Mechanism) 进行身份验证, 这也是 MongoDB 默认的身份验证机制.

### 使用SCARM进行身份验证
在前面的步骤中, MongoDB 容器已经启动了, 接着按以下步骤操作:
1. 在命令行中执行 `docker exec -it mongodb /bin/bash` 进入 MongoDB 容器中.
2. 进行容器内部的 /bin 目录下, 执行 `mongosh --port 27017` 连接到 MongoDB.
3. 连接到 MongoDB 后, 默认应该处于 test 数据库下, 执行 `use admin` 来切换到 admin 数据库. 切换后执行以下命令:
   ```js
   db.createUser(
        {
            user: "myUserAdmin",
            pwd: passwordPrompt(), // or cleartext password
            roles: [
                { role: "userAdminAnyDatabase", db: "admin" },
                { role: "readWriteAnyDatabase", db: "admin" }
            ]
        }
   )
   ```
   执行后会要求输入密码, 输入后就会创建一个名为 myUserAdmin 的用户, 你可以把他视作管理员用户, 待会我们需要用它来连接到 MongoDB, 并且创建其他用户.
4. 执行 `show users`, 正常情况下你就能看到刚刚创建好的用户. 确认用户已经创建好了, 接着退出 MongoDB 容器. 
5. 回到宿主机上, 先执行 `docker stop mongodb` 停止 MongoDB 容器. 接着进入到 MongoDB 配置文件所在目录下, 修改配置文件如下:
    ```yaml
    systemLog:
       destination: file
       path: "/var/mongodb/log/mongod.log"
       logAppend: true
    storage:
       dbPath: "/var/mongodb/data/db"
       journal:
          enabled: true
    net:
       bindIp: 127.0.0.1
       port: 27017
    security:
       # 开启身份认证
       authorization: enabled
    setParameter:
       enableLocalhostAuthBypass: false
    ```
6. 执行 `docker restart mongodb` 启动容器. 重复步骤 2 的操作, 这时执行 `show dbs` 来查看数据库列表, MongoDB 会报错并且提示需要认证, 证明身份认证已经开启了.
7. 退出 MongoDB 的连接, 回到容器内. 这一次执行:
   ```shell
    mongosh --port 27017 --authenticationDatabase "admin" -u "myUserAdmin"-p
   ```
   输入密码后就能连接到 MongoDB 中, 并且这一次执行 `show dbs` 就不会报错了.

#### 创建用户
一般来说, 上面我们创建的用户不应该直接给应用使用, 而是用它来为应用创建用户. 这些用户的权限限制到具体某个数据库或者表, 以便于提高安全性.

创建用户可以参考官网教程: https://www.mongodb.com/docs/manual/tutorial/create-users/

创建用户主要是使用 `db.createUser()` 方法, 下面是官网的一个例子:
```
db.createUser(
  {
    user: "myTester",
    pwd:  passwordPrompt(),   // or cleartext password
    roles: [ { role: "readWrite", db: "test" },
             { role: "read", db: "reporting" } ]
  }
)
```
我们主要关注的是 `roles` 数组里面权限项, role 代表具体权限项比如: read(读), readWrite(读和写). db 代表 role 生效的数据库. 上面的例子的意思是创建一个用户, 该用户拥有在 test 数据库中读和写的权限, 以及在 reporting 数据库中读的权限.

