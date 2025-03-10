**NFS的卷存储类型支持：**  
数据存储在远程 NFS 服务器上，Pod 删除后数据仍然保留。  
支持跨节点访问，适合多 Pod 共享数据  

官方参考文档:[配置Pod以使用PersistentVolume作为存储|Kubernetes](https://kubernetes.io/zh-cn/docs/tasks/configure-pod-container/configure-persistent-volume-storage/)  
**这里存储类型使用nfs动态存储，首先得安装nfs**，  
**目的：利用主节点作为存储结点（即NFS服务器），实现所有node的文件共享 并用于资源的配置中**  
**1.在所有节点上安装nfs**  
`yum install -y nfs-utils rpcbind  `  
**2.在nfs主节点上**  
创建共享目录  
`mkdir /nfs/data/01`  
`mkdir /nfs/data/02`  
编辑/etc/exports    
`vim /etc/exports`  
添加一下内容  
```
/nfs/data/01 *(rw,insecure,sync,no_root_squash)
/nfs/data/02 *(rw,insecure,sync,no_root_squash)
```

```
命令解释
/nfs/data/01:
这是 NFS 服务器上要共享的目录路径。
客户端可以挂载此目录并访问其中的文件。

*:
表示允许所有客户端访问该共享目录。
如果需要限制访问，可以将 * 替换为具体的 IP 地址或网段。如/nfs/data 192.168.219.0/24(insecure,rw,sync,no_root_squash)  这表示192.168.219网段下面的所有结点都可以共享该NFS文件夹
insecure:
允许客户端使用非特权端口（大于 1024 的端口）连接 NFS 服务器。
通常用于兼容性，但可能降低安全性。
rw:
允许客户端对共享目录进行读写操作。
如果设置为 ro，则客户端只能读取，不能写入。

sync:
确保数据在写入时同步到磁盘，保证数据一致性。
与 async 相对，async 会延迟写入以提高性能，但可能增加数据丢失的风险。

no_root_squash:
允许客户端的 root 用户保留 root 权限访问共享目录。
如果未设置此选项，客户端的 root 用户会被映射为 NFS 服务器上的 nobody 用户，权限受限。
注意：启用 no_root_squash 可能存在安全风险，因为客户端的 root 用户可以对共享目录进行任意操作。
```
**启动服务**
`systemctl start rpcbind`
`systemctl start nfs`

**设置开启启动**
`systemctl enable rpcbind`
`systemctl enable nfs`


**配置生效**
`exportfs -r`

**3.在从节点上**
快速验证 NFS 配置是否正确  
`showmount -e 主节点ip`  
创建与NFS服务器（ 192.168.219.142）相同的目录，并执行命令将NFS服务器的目录挂载到本地节点  
`mkdir -p /nfs/data/01`  
`mkdir -p /nfs/data/02`  
`mount -t nfs 192.168.219.142:/nfs/data /nfs/data`  
