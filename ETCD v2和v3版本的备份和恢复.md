#  背景

生成准备上k8s,为了保证之前swarm网络和k8s网络相同.就要使用同一个ETCD.在之前etcd是http协议单节点,现在就验证一下使用同一个etcd,是否会产生脏数据或者丢数据.

# 步骤

## 重要

由于etcd v2和v3基本上是两个割裂的版本.造成数据相互不可见.使用的API版本不同,备份和恢复的步骤也是千差万别.还有个就是很重要的地方

- **若使用 v3 备份数据时存在 v2 的数据则不影响恢复**
- **若使用 v2 备份数据时存在 v3 的数据则恢复失败**

开始吧!

# 查看API版本

```shell
[root@k8s-master soft]# etcdctl --version
etcdctl version: 3.3.11
API version: 2
```

这里可以看到API版本是2,如果要使用API 3的命令,记住在命令前面加入`ETCDCTL_API=3`.

>  flannel使用的是v2版本的接口写入的数据,那么我们就用v2版本的命令去备份数据

# 备份

> V2版本

```shell
etcdctl backup --data-dir /var/lib/etcd/default.etcd --backup-dir /tmp/etcd_backup
```

> v3版本

```shell
[root@k8s-master1 ~]# ETCDCTL_API=3 etcdctl --endpoints 127.0.0.1:2379 snapshot save snashot.db
Snapshot saved at snashot.db
```

# 恢复

> v2版本

打包之前v2命令备份的文件夹,解压到新的etcd节点的数据目录

- 运行ETCD

```shell
etcd --data-dir=/var/lib/etcd/default.etcd --force-new-cluster &
```

- 查看ID

```shell
[root@k8s-master soft]# etcdctl member list
7c26cff88201: name=default peerURLs=http://127.0.0.1:2379 clientURLs=http://0.0.0.0:4001,http://172.10.4.87:2379 isLeader=true
```

- 数据同步

```shell
curl http://127.0.0.1:2379/v2/members/7c26cff88201 -XPUT \
-H "Content-Type:application/json" -d '{"peerURLs":["http://127.0.0.1:2379"]}'
```



执行完之后,kill掉ETCD

# 重新启动服务

```shell
systemctl start etcd
```

如果不报错的话,应该问题不大

# 查看数据

这里推荐一个etcd的webui工具etcdkeeper

docker安装一下

```shell
docker run -it -d -p 8080:8080 --name etcdkeeper evildecay/etcdkeeper
```

启动之后,登录界面,我这里使用的是9090端口,因为这台机器上面有服务使用8080端口了.

![image-20210928093624340](D:\1_WORK\markdown笔记\sre\assets\image-20210928093624340.png)

关于V3数据的恢复,还没有试过.不过可以查看这两篇博文

https://doczhcn.gitbook.io/etcd/index/index-1/recovery



V2版本备份参考博文

https://www.cnblogs.com/ityunv/p/9173081.html
