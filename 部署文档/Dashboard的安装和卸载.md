### 建议看原文链接：https://blog.csdn.net/Dandelin/article/details/138296061

以下内容未整理：
1.因为安装dashboard需要helm命令，因此首先安装helm至master节点 Releases · helm/helm · GitHub 这里网上都有教程不做阐述

2.(k8s忽略直接下一步）

helm会使用kubectl默认的KUBECONFIG配置，这里我们需要将KUBECONFIG换成k3s的否则会链接失败。在master主机上输入命令

export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
3.helm下载完毕后，根据dashboard官网GitHub - kubernetes/dashboard: General-purpose web UI for Kubernetes clusters

在master主机中添加dashboard源

helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard/
之后我们直接进行一个更新

helm upgrade --install kubernetes-dashboard kubernetes-dashboard/kubernetes-dashboard --create-namespace --namespace kubernetes-dashboard
会出现以下结果

但是我们发现外部网站并不能访问dashboard

4.查看命名空间kubernetes-dashboard下的服务

kubectl -n kubernetes-dashboard get svc
与网上常规的只有一个kubernetes-dashboard服务不同 这里有多个kubernetes-dashboard服务

网上并没有详细说明，经过测试我发现 我们只需对kubernetes-dashboard-kong-proxy的type进行修改

5.将kubernetes-dashboard-kong-proxy的TYPE从ClusterIp修改为NodePort这样dashboard才能提供外部的访问端口

kubectl -n kubernetes-dashboard edit svc kubernetes-dashboard-kong-proxy


6.在任何地方新建的一个yaml名称为dashboard-admin.yaml

vi dashboard-admin.yaml
内容为：
```
# vim dashboard.admin-user.yaml
# 创建ServiceAccount
apiVersion: v1
kind: ServiceAccount
metadata:
 name: admin-user
 namespace: kubernetes-dashboard
---
#创建clusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
 name: admin-user
roleRef:
 apiGroup: rbac.authorization.k8s.io
 kind: ClusterRole
 name: cluster-admin
subjects:
 - kind: ServiceAccount
   name: admin-user
   namespace: kubernetes-dashboard
```
7.之后我们运行如下指令：

kubectl apply -f dashboard-admin.yaml
8.

1.24之后Kubernetes不再为ServiceAccound自动生成Secret，因此之后需要tocken使用如下方式创建：

kubectl -n kubernetes-dashboard create token admin-user


9，查看更改后 Dashboard向外部提供的访问端口



在外部浏览器输入



注意红框是你自己的master主机的IP地址，可以用ip addr 查看 ens33的ip地址即是

10.

第一次访问需要点高级 选择继续访问，然后复制之前提供的token粘贴上去 即可登录dashboard

### 卸载
因为是用helm安装的 所以用它来卸载
helm list -n kubernetes-dashboard
helm uninstall <release-name> -n kubernetes-dashboard
将 <release-name> 替换为实际的 Helm Release 名称。
删除当前的 ServiceAccount 和 ClusterRoleBinding
kubectl get serviceaccount -n kubernetes-dashboard
kubectl get clusterrolebinding

使用以下命令确认资源已被删除。

kubectl delete serviceaccount admin-user -n kubernetes-dashboard
kubectl delete clusterrolebinding admin-user
确保 admin-user 不在列表中


1. . 检查 Kubernetes Dashboard 资源是否已删除
kubectl get all -n kubernetes-dashboard
2. 删除 kubernetes-dashboard 命名空间
kubectl delete namespace kubernetes-dashboard

3. 验证命名空间删除情况

kubectl get namespaces

4. 验证清理结果
kubectl get all --all-namespaces | grep dashboard
如果没有输出，说明所有与 Kubernetes Dashboard 相关的资源都已成功删除

5.将k8s资源从 helm仓库移除
helm repo remove kubernetes-dashboard
