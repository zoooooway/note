# CentOS 搭建NFS

> 使用阿里云服务器，CentOS 8
>
> 参考：
>
> http://linux.vbird.org/linux_server/0330nfs.php
>
> https://zhuanlan.zhihu.com/p/81752517
>
> https://www.cnblogs.com/zeq912/p/9606105.html
>
> http://www.myjishu.com/?p=342
>
> https://www.jianshu.com/p/e842d67c1eac
>
> http://linux.vbird.org/linux_server/0250simple_firewall.php

## 服务端

* ### 关闭防火墙及selinux

  因为我使用阿里云的云服务器，带有安全组，所以直接关闭了服务器的防火墙。若想保留防火墙，则可跳过此步骤，并完成下面固定端口和安全组的做法后，将所需端口放行。

  * #### 防火墙

    ```shell
    systemctl disable firewalld
    ```

  * #### selinux

    ```shell
    systemctl stop firewalld
    ```

* ### 安装NFS

  ```shell
  yum -y install nfs-utils
  ```

* ### 配置共享目录信息

  ```shell
  [root@nfs ~]# echo '/data_server 172.16.1.0/24(rw,sync,all_squash)' > /etc/exports
  
  [root@nfs ~]# cat /etc/exports
  
  /data_server 172.16.1.0/24(rw,sync,all_squash) # 172.16.1.0-24网段内的用户拥有读写权限，且都以匿名用户访问
  ```

> 执行man exports命令，然后切换到文件结尾，可以快速查看如下样例格式：
>
> | 共享参数       | 参数作用                                                     |
> | -------------- | ------------------------------------------------------------ |
> | rw*            | 读写权限                                                     |
> | ro             | 只读权限                                                     |
> | root_squash    | 当NFS客户端以root管理员访问时，映射为NFS服务器的匿名用户(不常用) |
> | no_root_squash | 当NFS客户端以root管理员访问时，映射为NFS服务器的root管理员(不常用) |
> | all_squash     | 无论NFS客户端使用什么账户访问，均映射为NFS服务器的匿名用户(常用) |
> | no_all_squash  | 无论NFS客户端使用什么账户访问，都不进行压缩                  |
> | sync*          | 同时将数据写入到内存与硬盘中，保证不丢失数据                 |
> | async          | 优先将数据保存到内存，然后再写入硬盘；这样效率更高，但可能会丢失数据 |
> | anonuid*       | 配置all_squash使用,指定NFS的用户UID,必须存在系统             |
> | anongid*       | 配置all_squash使用,指定NFS的用户UID,必须存在系统             |

* ### 创建共享目录，设置权限

  * #### 创建共享目录

    ```shell
    mkdir /data_server
    ```

    如要使用已存在目录进行挂载，则跳过此步骤

  * #### 设置目录权限

    ```shell
    chmod 777 /data_server # 设置所有用户可进行读写
    ```

* ### 配置固定端口

  NFS服务启动时会随机使用端口向RPC服务进行注册，阿里云有安全组的设定，不配置固定端口的话每次NFS服务重启都需要更改安全组配置。如不需要固定端口可跳过此节。

  * #### 配置NFS端口

    网上很多配置固定端口的说法都差不多，需要修改两个文件配置。问题是他们说的文件我找不到...，我猜可能是版本不一样吧。依据我自己尝试了一下，可以修改这几个文件来配置：

    第一个是：*/etc/nfs.conf* 。将以下port的属性都打开，且改为固定值

    ```properties
      ......
      [lockd]
      # 将此处注释打开，下同
       port=30002
       udp-port=30002
      #
      [mountd]
      # debug=0
      # manage-gids=n
      # descriptors=0
       port=30003
      # threads=1
      # reverse-lookup=n
      # state-directory-path=/var/lib/nfs
      # ha-callout=
      #
      [nfsdcld]
      # debug=0
      # storagedir=/var/lib/nfs/nfsdcld
      #
      [nfsdcltrack]
      # debug=0
      # storagedir=/var/lib/nfs/nfsdcltrack
      #
      [nfsd]
      # debug=0
      # threads=8
      # host=
       port=30006 
      # grace-time=90
      # lease-time=90
      # tcp=y
      # vers2=n
      # vers3=y
      # vers4=y
      # vers4.0=y
      # vers4.1=y
      # vers4.2=y
      # rdma=n
      # rdma-port=20049
      #
      [statd]
      # debug=0
       port=30004 
      # outgoing-port=0
      # name=
      # state-directory-path=/var/lib/nfs/statd
      # ha-callout=
      # no-notify=0
      ......
    ```
    
    修改这个文件后启动NFS（见下节）并执行：
    
    ```shell
    rpcinfo -p
    ```
    
    会发现`nlockmgr`这个服务的端口并不是上面修改的值（30002），这时候就要执行以下命令：
    
    ```shell
    cp /etc/sysctl.conf /etc/sysctl.conf.$(date +%F)
    sed -i '$a fs.nfs.nlm_tcpport=30002\nfs.nfs.nlm_udpport=30002' /etc/sysctl.conf # 设置nlockmgr服务端口为30002
    sysctl -p # 刷新配置
    ```
    
    > 注意这个端口值不要和上面 */etc/nfs.conf*文件的标签下的除 [lockd]下的其他port使用相同值，否则无法启动NFS
    

* ### 启动NFS

  * #### 启动并设置开机自启

    ```shell
    # NFS依赖RPC服务
    systemctl enable rpcbind
    systemctl start rpcbind
    # NFS
    systemctl enable nfs-server
    systemctl start nfs-server 
    ```

  * #### 查看端口

    ```shell
    rpcinfo -p
    ```

  * #### 检查挂载是否生效

    ```shell
    [root@nfs ~]# showmount -e
    # 不生效则重启nfs与rpc服务后再试
    [root@nfs ~]# systemctl restart rpcbind
    [root@nfs ~]# systemctl restart nfs-server
    ```

* ## 配置安全组规则

  如果是阿里云或者其他带有安全组的云服务器，则需要配置一下安全组的规则，将NFS使用到的端口放行。

  可以使用`rpcinfo -p`来查看使用的端口（区分`TCP`和`UDP`）。

  即上面设置的固定端口都需要放行，主要`TCP`和`UDP`需要分开放行。除了设置的几个固定端口，还需要放行以下端口：
  
  * `udp` 111
  * `tcp` 111
  * `udp ` 4046
  * `tcp` 2049
  
  > 参考：https://blog.csdn.net/fhqsse220/article/details/45668057?utm_medium=distribute.pc_aggpage_search_result.none-task-blog-2~aggregatepage~first_rank_ecpm_v1~rank_aggregation-1-45668057.pc_agg_rank_aggregation&utm_term=nfs%E6%8C%82%E8%BD%BD%E9%9C%80%E8%A6%81%E5%BC%80%E9%80%9A%E7%9A%84%E7%AB%AF%E5%8F%A3&spm=1000.2123.3001.4430
  >
  > 如果不放行这些端口，会在客户端挂载时: 使用 `mount 10.12.13.11:/vol/lft_jjmk /mnt` 报错：*mount.nfs: Connection timed out*
  >
  > 折腾了好久好久好久好久......
  

## 客户端

* ### 安装NFS

  同服务端

* ### 挂载目录

  * #### 查看远程服务器提供的挂载信息

    ```shell
    showmount -e 172.16.1.31
    ```

  * #### 创建挂载点目录，执行挂载命令

    ```shell
    # data_local为客户端要挂载到的目录路径
    mount -t nfs 172.16.1.31:/data_server /data_local 
    ```

  * #### 测试是否可读写

    ```shell
    echo "123" > /data_local/test.txt
    ```

* ### 永久挂载NFS

  > 生产环境不建议，如果服务器异常会导致客户端启动不起来

  如果希望NFS文件共享服务能一直有效，则需要将其写入到fstab文件中

  ```shell
  #编辑fstab文件
  [root@nfs-client ~]# vim /etc/fstab
  172.16.1.31:/data_server /data_local nfs defaults 0 0
  #验证fstab是否书写正确
  [root@nfs-client ~]# mount -a
  ```

* ### 卸载NFS

  ```shell
  [root@nfs-client ~]# umount /nfsdir 
  #注意:卸载的时候如果提示”umount.nfs: /nfsdir: device is busy” 
  #1.切换至其他目录, 然后在进行卸载。
  #2.NFS Server宕机, 强制卸载umount -lf /nfsdir
  ```

