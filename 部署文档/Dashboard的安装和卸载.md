### 建议看原文链接：https://blog.csdn.net/Dandelin/article/details/138296061

以下内容未整理：
1.因为安装dashboard需要helm命令，因此首先安装helm至master节点 [Releases · helm/helm · GitHub](https://github.com/helm/helm/releases) 这里网上都有教程不做阐述

2.(k8s忽略直接下一步）

helm会使用kubectl默认的KUBECONFIG配置，这里我们需要将KUBECONFIG换成k3s的否则会链接失败。在master主机上输入命令

`export KUBECONFIG=/etc/rancher/k3s/k3s.yaml`
3.helm下载完毕后，根据dashboard官网[GitHub - kubernetes/dashboard: General-purpose web UI for Kubernetes clusters](https://github.com/kubernetes/dashboard)

在master主机中添加dashboard源

`helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard/`
之后我们直接进行一个更新

`helm upgrade --install kubernetes-dashboard kubernetes-dashboard/kubernetes-dashboard --create-namespace --namespace kubernetes-dashboard`

下面步骤按照
[在K8s_K3s集群中安装与卸载Dashboard_k3s dashboard-CSDN博客.pdf](https://github.com/user-attachments/files/19214368/K8s_K3s.Dashboard_k3s.dashboard-CSDN.pdf)
