# MongoDB

创建管理员用户

``` bash
db.createUser({user:"root", pwd:"password",roles:[{role:"root", db:"admin"}]})
```

更改密码：

``` bash
db.changeUserPassword('user1','userpwd');  
```

## 通过docker迁移mongodb


* 数据备份命令：
``` bash
mongodump --host ip:port -u mongouser -p thepassword --authenticationDatabase=admin --db=testdb -o /data/dump_testdb
```

* 数据备份还原命令:
> 进入mongodb容器内运行
``` bash
mongorestore --host ip:port -u mongouser -p thepassword --authenticationDatabase=admin --dir=/data/dump_testdb
```

### 常见异常

* 备份命令使用的用户没有足够的权限
``` log
Failed: error getting collections for database `admin`: error running `listCollections`. Database: `admin` Err: not authorized on admin to execute command {lcursor: {}, $readPreference: { mode: "secondaryPreferred" }, $db: "admin" }
```

**解决办法**
``` bash
> use admin
 
> db.auth("root","testpassword")
 
> db.grantRolesToUser ( "root", [ { role: "__system", db: "admin" } ] )  #授权给admin用户对system.version表执行命令的权限
```