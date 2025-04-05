报错："https://192.168.219.129:80/v2/library/jenkins-maven/manifests/v1": http: server gave HTTP response to HTTPS client
这个错误表明 Kubernetes 在尝试从你的私有镜像仓库 192.168.219.129:80 拉取镜像 jenkins-maven:v1 时失败了，具体原因是 HTTP/HTTPS 协议不匹配

这是因为k8s的容器运行时可能是containerd  

## 如果容器运行时是docker  
则只需要在`/etc/daemon.json`中配置`"insecure-registries": ["192.168.219.129:80"]` 就可以使用Http协议访问   

## 如果是containerd则配置：  
`vim /etc/containerd/config.toml`  
```yaml
[plugins."io.containerd.grpc.v1.cri".registry]
  config_path = ""  # 保留空字符串（或删除此行）

  [plugins."io.containerd.grpc.v1.cri".registry.mirrors]
    [plugins."io.containerd.grpc.v1.cri".registry.mirrors."192.168.219.129:80"] #新增,仓库地址
      endpoint = ["http://192.168.219.129:80"]  # 新增,显式指定 HTTP 协议

  [plugins."io.containerd.grpc.v1.cri".registry.configs]
    [plugins."io.containerd.grpc.v1.cri".registry.configs."192.168.219.129:80".tls] #新增
      insecure_skip_verify = true  # 新增，关键： Harbor 使用 HTTP，必须配置来跳过TLS证书验证

  # 以下为空区块可删除（除非需要特殊配置）
  [plugins."io.containerd.grpc.v1.cri".registry.auths]
  [plugins."io.containerd.grpc.v1.cri".registry.headers]
```
重启containerd `systemctl restart containerd` 
