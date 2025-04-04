这里使用的是带有maven环境和Sonarqube-cli工具的 jenkins镜像进行安装，首先要构建这个镜像：[Jenkins镜像构建并推送到harbor仓库](https://github.com/Zorinman/linux-docker-k8s/blob/main/docker/%E9%85%8D%E7%BD%AE%E4%B8%8E%E6%93%8D%E4%BD%9C/Dockerfile%E6%9E%84%E5%BB%BA%E9%95%9C%E5%83%8F.md)  

这里使用了jenkins-configmap.yaml  jenkins-pvc.yaml     jenkins-service.yaml  jenkins-deployment.yaml  jenkins-serviceAccount.yaml,

## 一、首先需要对jenkins-deployment.yaml作修改:  
1.拉取的Jenkins镜像修改为之前构建完成并存在Harbor仓库的镜像 ：192.168.219.129:80/library/jenkins-maven:v1    
2.volumeMounts中的mvn-settings的mountPath修改为3.9.9  

## 二、如果k8s的容器运行时是containerd 则需要为containerd配置以便于k8s从harbor仓库拉取镜像     

(类似于在为docker在/etc/daemon.json中配置"insecure-registries": ["192.168.219.129:80"]  
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
这里jenkins-pvc使用的是之前配置好的nfs制备器自动创建PV  
安装  
`kubectl apply -f . `
查看 service 端口，通过浏览器访问  
 `kubectl get svc -n kube-devops `
查看容器日志，搜索关键词password获取默认登录密码    
`kubectl logs -f <podname> -n kube-devops`
这里我的是53968408369d4fc99bd7c7c1a7d8adb1   
登录之后我将 用户名和密码改为了 admin 和123  
