### 1.如果k8s的运行时是docker 
设置yaml文件镜像拉取策略为IfNotPresent
```
  ...
    name: elasticsearch-logging-init
    image: alpine:3.6
    imagePullPolicy: IfNotPresent
  ...
```
在资源要部署的节点上使用docker拉取镜像
`docker pull <yaml文件里的镜像名字>`
应用yaml文件


### 2.如果k8s的运行时是containerd
 使用docker拉取镜像之后将 Docker 镜像保存为 .tar 文件
 `docker save -o alpine-3.6.tar alpine:3.6`
 使用 ctr 导入镜像到 containerd
 `sudo ctr -n=k8s.io images import alpine-3.6.tar`
 验证镜像是否导入成功
` sudo ctr -n=k8s.io images list`
应用yaml文件
