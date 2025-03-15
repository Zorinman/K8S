k8s 容器运行时为 containerd，部署 pod到node1时发现不能自动拉取镜像， 但是在部署到的节点上用 containerd 拉取时发现可以手动拉取下来  
如果开起了全局代理，首先确定 containerd是否已经单独配置了代理  
解决:  
1.（解决）可能是 containerd 与 k8s 状态不同步，在 node1 上重启 containerd
systemctl daemon-reload
systemctl restart containerd
可以尝试在 master 上也进行重启

2.（临时解决）在 node1 上把 pod 的 yaml 中需要的 image 使用 containerd 一个个手动拉取下来 yaml 中的镜像拉取策略设置为优先从本地拉取，这样可以临时解决而且要一个个拉取镜像非常麻烦。
