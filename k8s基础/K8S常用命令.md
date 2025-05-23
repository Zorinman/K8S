**查看configmap**：`kubectl get cm`

**实时查看Pod的默认容器的日志**：`kubectl logs -f <pod名称>`

**如果 Pod 中有多个容器，使用 -c 指定容器名称**：`kubectl logs -f <pod名称> -c <容器名称>`


**查看某个命名空间下的所有资源**：`kubectl get all -n <namespace>`

查**看特定名称空间下的 服务**:`/pod/deploy：kubectl -n xxx get svc/pod/deploy`

**使yaml文件生效**：`kubectl apply -f xxx.yaml` 

**查看所有Pod以及它们所在的节点**：`kubectl get pods -o wide -A`  
**详细查看pod里的内容（可以查看Pod内的容器）** `kubectl describe pods/podname`  

**选择pod中的特定容器下执行命令**：`kubectl exec -it <pod-name> -c <container-name> -- <command>`

**创建一个交互式终端会话连接到容器**：`kubectl exec -it my-pod -c container2 -- /bin/sh`

**如果不指定容器-c 则表示直接在pod的默认容器执行**：`kubectl exec -it <pod-name> -- <command>`

**删除Pod service depoly**  
`kubectl delete deploy XX`  
`kubectl delete pod   XX`  
`kubectl delete svc xx`  
先删deploy 再删pod 否则pod会自己创建  

**删除一个yaml文件里定义的所有资源（⭐如果使用了PVC持久化存储先删除PV和PVC）**：`kubectl delete  -f    XXX.yaml`  

**删除当前文件夹下面所有yaml文件定义的资源（⭐如果使用了PVC持久化存储先删除PV和PVC）**!  
` pwd查看路径`   
` kubectl delete  -f <目录路径>`   

**重启对应deployment的pod：**  
例重启coredns的pod:kubectl rollout restart deployment -n kube-system coredns  



![image](https://github.com/user-attachments/assets/5e31a056-978d-432a-b2ec-f9eb1fba7075)


如果pod处于Terminating无法删除：    
⭐可能是使用了持久化存储 先删除对应PV和PVC  
`kubectl delete pvc <pvc-name>`  

还是不行则强制删除：  
`kubectl delete pod <pod_name> --force --grace-period=0`

例：`kubectl delete pod mysql-dep-74d9b967f6-rztck --force --grace-period=0`

![image](https://github.com/user-attachments/assets/83224734-3255-4bc1-b5cd-af5518d869e3)
![image](https://github.com/user-attachments/assets/b95198c5-2c60-4cc4-b614-bc783d88af2a)

### /bin/sh 和 /bin/bash的区别：
/bin/sh 和 /bin/bash 都是 Unix/Linux 系统中的 Shell 解释器  
sh 是 Bourne Shell 的缩写，是 Unix 系统中最经典的 Shell 解释器。  
bash 是 Bourne-Again Shell 的缩写，是 sh 的增强版。  
在容器中，/bin/sh 更常见，因为容器通常追求最小化，而 sh 的体积更小  
果你希望脚本能在所有 Unix/Linux 系统上运行，应该使用 #!/bin/sh（即使用 sh 语法）。  
如果你只需要在支持 bash 的系统上运行脚本，可以使用 #!/bin/bash  
