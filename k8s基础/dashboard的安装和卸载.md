## dashboard安装:
https://blog.csdn.net/Dandelin/article/details/138296061
获取token命令：  
`kubectl -n kubernetes-dashboard create token admin-user`
## dashboard卸载:  
因为是用helm安装的 所以用它来卸载    
`helm list -n kubernetes-dashboard`  
`helm uninstall <release-name> -n kubernetes-dashboard`  
将 <release-name> 替换为实际的 Helm Release 名称。  
删除当前的 ServiceAccount 和 ClusterRoleBinding  
`kubectl get serviceaccount -n kubernetes-dashboard`  
`kubectl get clusterrolebinding`  
 
使用以下命令确认资源已被删除。  

`kubectl delete serviceaccount admin-user -n kubernetes-dashboard`  
`kubectl delete clusterrolebinding admin-user`  
确保 admin-user 不在列表中  


1.  检查 Kubernetes Dashboard 资源是否已删除  
`kubectl get all -n kubernetes-dashboard`  
2. 删除 kubernetes-dashboard 命名空间  
`kubectl delete namespace kubernetes-dashboard`  

3. 验证命名空间删除情况  

`kubectl get namespaces`  

4. 验证清理结果  
`kubectl get all --all-namespaces | grep dashboard`  
如果没有输出，说明所有与 Kubernetes Dashboard 相关的资源都已成功删除  

5.将k8s资源从 helm仓库移除  
`helm repo remove kubernetes-dashboard`  



