[NFS的安装](https://github.com/Zorinman/K8S/blob/main/%E9%83%A8%E7%BD%B2%E6%96%87%E6%A1%A3/NFS%E5%AE%89%E8%A3%85.md)  


![image](https://github.com/user-attachments/assets/479c94cd-7b5c-4c4b-880d-14a33659dc08)
```
 1. hostPath
定义: hostPath 卷将主机节点上的文件或目录挂载到 Pod 中。

用途: 通常用于访问主机上的特定文件或目录，如日志文件、配置文件等。

特点:

数据存储在主机节点上，Pod 删除后数据仍然保留。

不具备跨节点的可移植性，Pod 调度到其他节点时无法访问相同数据。

可能存在安全隐患，因为 Pod 可以直接访问主机文件系统。
```
```
2. emptyDir
定义: emptyDir 卷是一个临时目录，随 Pod 创建而创建，随 Pod 删除而删除。

用途: 用于 一个Pod 内多个容器之间的数据共享或临时存储。

特点:

数据存储在节点上，生命周期与 Pod 绑定。

Pod 删除后，数据也会被清除。

适合临时存储，不具备持久性。
```
```
3.NFS (Network File System)
定义: NFS 卷将远程 NFS 服务器（例如专门用来做存储的服务器)的上的目录挂载到 Pod 中。

用途: 用于需要持久化存储且跨node共享数据的场景。

特点:

数据存储在远程 NFS 服务器上，Pod 删除后数据仍然保留。

支持跨node访问，适合多 Pod 共享数据。

需要提前配置 NFS 服务器。
```
