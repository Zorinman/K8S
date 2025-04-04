# Kubernetes 镜像缓存问题及解决方案

## 问题描述

尽管执行了以下操作：

1. 清除了持久化存储下的所有文件
   ```yaml
   name: jenkins-data 
   persistentVolumeClaim:
       claimName: jenkins-pvc
   ```
2. 删除了PV、PVC资源
3. 删除了pod、deployment、service 所有资源
4. Harbor仓库删除了旧的镜像
5. 本地docker和containerd确认没有旧镜像

但是，重新执行 `kubectl apply -f .` 之后发现：
- Pod内的日志还是之前的日志信息
- 并没有从Harbor仓库上拉取新镜像，还是使用旧的镜像

## 原因分析

这是由于Kubernetes本地缓存的镜像，即使删除了本地的Docker镜像或containerd镜像，Kubernetes依然可能在本地节点上缓存了某些镜像。

## 解决方案

### 强制拉取镜像

可以通过设置或修改 `imagePullPolicy` 为 `Always` 来强制Kubernetes每次都拉取最新的镜像。

例如，更新YAML文件中的imagePullPolicy：

```yaml
containers:
  - name: jenkins
    image: 192.168.219.129:80/library/jenkins-maven:v1
    imagePullPolicy: Always
```

⭐成功拉取之后 记得kubectl edit deploymen  把imagePullPolicy改为IfNotPresent，否则每次都会尝试自动从镜像地址拉取镜像.  
