# MongoDB 备份脚本

根据需求修改以下内容，并保存为 *.bat* 文件，并放在`MongoDB`安装目录下的 *bin* 文件夹下。

```shell
@echo off
set "Ymd=%date:~,4%%date:~5,2%%date:~8,2%"
set port=27017
set user=user
set pass=pass
set host=127.0.0.1
set dbname=watersupervisionImg
set backupfile=.\mongodump\%Ymd%\
mongodump -h %host% --port %port% -u=%user% -p=%pass% -d %dbname% -o %backupfile%
```

可先双击执行，查看是否有备份成功。

然后使用`windows`计划程序添加定时任务即可。（详见 *数据库逻辑备份方案.docx* ）

