# Centos8安装docker

将以下内容保存成 `dockerIntall.sh` 并执行 `sh dockerIntall.sh`

```shell
\#!/bin/bash 

# 移除掉旧的版本 
sudo yum remove docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate   docker-selinux docker-engine-selinux docker-engine 

# 删除所有旧的数据 
sudo rm -rf /var/lib/docker 

#  安装依赖包 
sudo yum install -y yum-utils device-mapper-persistent-data lvm2 

# 添加源，使用了阿里云镜像 

sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo 

# 配置缓存 
sudo yum makecache fast 

# 安装最新稳定版本的docker 
sudo yum install docker-ce docker-ce-cli containerd.io 

# 启动docker引擎并设置开机启动 
sudo systemctl start docker 
sudo systemctl enable docker 

#测试 
docker run hello-world  
```



