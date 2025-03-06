**查看某个命名空间下的所有资源**：`kubectl get all -n <namespace>`

查**看特定名称空间下的 服务**:`/pod/deploy：kubectl -n xxx get svc/pod/deploy`

**使yaml文件生效**：`kubectl apply -f xxx.yaml` 

**查看所有Pod以及它们所在的节点**：`kubectl get pods -o wide -A`  
**详细查看pod里的内容（可以查看Pod内的容器）** `kubectl describe pods/podname`  

**选择pod中的特定容器下执行命令**：`kubectl exec -it <pod-name> -c <container-name> -- <command>`

**创建一个交互式终端会话连接到容器**：`kubectl exec -it my-pod -c container2 -- /bin/sh`

**如果不指定容器-c 则表示直接在pod的默认容器执行**：`kubectl exec -it <pod-name> -- <command>`

![image](https://github.com/user-attachments/assets/6b731fee-a13c-457b-bc52-cf70caef575b)  
![image](https://github.com/user-attachments/assets/5e31a056-978d-432a-b2ec-f9eb1fba7075)


如果pod处于Terminating无法删除可以强制删除：  
强制删除：  
kubectl delete pod <pod_name> --force --grace-period=0

kubectl delete pod mysql-dep-74d9b967f6-rztck --force --grace-period=0

![image](https://github.com/user-attachments/assets/83224734-3255-4bc1-b5cd-af5518d869e3)
![image](https://github.com/user-attachments/assets/b95198c5-2c60-4cc4-b614-bc783d88af2a)

### /bin/sh 和 /bin/bash的区别：
/bin/sh 和 /bin/bash 都是 Unix/Linux 系统中的 Shell 解释器  
sh 是 Bourne Shell 的缩写，是 Unix 系统中最经典的 Shell 解释器。  
bash 是 Bourne-Again Shell 的缩写，是 sh 的增强版。  
在容器中，/bin/sh 更常见，因为容器通常追求最小化，而 sh 的体积更小  
果你希望脚本能在所有 Unix/Linux 系统上运行，应该使用 #!/bin/sh（即使用 sh 语法）。  
如果你只需要在支持 bash 的系统上运行脚本，可以使用 #!/bin/bash  
