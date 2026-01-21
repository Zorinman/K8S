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

# 整理版本
# Kubernetes (K8s) 常用命令备忘录

## 1. Node 节点管理
| 描述                   | 命令                                                  |
| :--------------------- | :---------------------------------------------------- |
| **查看服务器节点**     | `kubectl get nodes`                                   |
| **查看服务器节点详情** | `kubectl get nodes -o wide`                           |
| **节点打标签**         | `kubectl label nodes <节点名称> labelName=<标签名称>` |
| **查看节点标签**       | `kubectl get node --show-labels`                      |
| **删除节点标签**       | `kubectl label node <节点名称> labelName-`            |

## 2. Namespace 命名空间
| 描述             | 命令                                          |
| :--------------- | :-------------------------------------------- |
| **查看命名空间** | `kubectl get namespace` (或 `kubectl get ns`) |
| **创建命名空间** | `kubectl create ns <名称>`                    |
| **删除命名空间** | `kubectl delete ns <名称>`                    |

## 3. Pod 管理
### 查看与排查
| 描述                                           | 命令                                           |
| :--------------------------------------------- | :--------------------------------------------- |
| **查看pod节点**                                | `kubectl get pod`                              |
| **查看所有pod节点**                            | `kubectl get pods -A`                          |
| **查看pod节点详情**                            | `kubectl get pod -o wide`                      |
| **查看所有名称空间下的pod**                    | `kubectl get pod --all-namespaces`             |
| **查看异常的pod节点**                          | `kubectl get pods -n <名称空间>`               | grep -v Running` |
| **查看异常pod节点的日志**                      | `kubectl describe pod <pod名称> -n <名称空间>` |
| **实时查看Pod的默认容器的日志**                | `kubectl logs -f <pod名称>`                    |
| **如果 Pod 中有多个容器，查看指定容器日志**    | `kubectl logs -f <pod名称> -c <容器名称>`      |
| **详细查看pod里的内容（可以查看Pod内的容器）** | `kubectl describe pods/podname`                |
| **查看所有Pod以及它们所在的节点**              | `kubectl get pods -o wide -A`                  |

### 创建与操作
| 描述                                               | 命令                                                           |
| :------------------------------------------------- | :------------------------------------------------------------- |
| **根据yaml文件创建pod**                            | `kubectl apply -f <文件名称>`                                  |
| **根据yaml文件删除pod**                            | `kubectl delete -f <文件名称>`                                 |
| **删除pod节点**                                    | `kubectl delete pod <pod名称> -n <名称空间>`                   |
| **普通方式创建pod**                                | `kubectl run <pod名称> --image=<镜像名称>`                     |
| **进入默认命名空间的pod节点**                      | `kubectl exec -it <pod名称> -- /bin/bash`                      |
| **进入某个特定命名空间下的pod节点**                | `kubectl exec -it <pod名称> -n <名称空间> -- /bin/bash`        |
| **选择pod中的特定容器下执行命令**                  | `kubectl exec -it <pod-name> -c <container-name> -- <command>` |
| **如果不指定容器-c 则表示直接在pod的默认容器执行** | `kubectl exec -it <pod-name> -- <command>`                     |
| **监控pod(一秒钟更新一次命令)**                    | `watch -n 1 kubectl get pod`                                   |
| **强制删除pod (pod显示Terminating无法删除)**       | `kubectl delete pod <pod_name> --force --grace-period=0`       |

## 4. Deployment 部署管理
### 创建与扩缩容
| 描述                                              | 命令                                                                  |
| :------------------------------------------------ | :-------------------------------------------------------------------- |
| **deployment部署pod(具有自愈能力，宕机自动拉起)** | `kubectl create deployment <pod名称> --image=<镜像名称>`              |
| **deployment部署pod(多副本)**                     | `kubectl create deployment <pod名称> --image=<镜像名称> --replicas=3` |
| **查看deployment部署**                            | `kubectl get deploy`                                                  |
| **删除deployment部署**                            | `kubectl delete deploy <pod名称>`                                     |
| **deployment扩容缩容pod**                         | `kubectl scale deploy/<pod名称> --replicas=<5>`                       |
| **deployment扩容缩容pod (编辑模式)**              | `kubectl edit deploy <pod名称>`                                       |

### 更新与回滚
| 描述                                            | 命令                                                                       |
| :---------------------------------------------- | :------------------------------------------------------------------------- |
| **deployment滚动更新pod**                       | `kubectl set image deploy/<pod名称> <容器名称>=<镜像名称:版本号> --record` |
| **deployment查看pod回退版本**                   | `kubectl rollout history deploy/<pod名称>`                                 |
| **deployment查看pod回退版本详情**               | `kubectl rollout history deploy/<pod名称> --revision=1`                    |
| **deployment回退pod到上一个版本**               | `kubectl rollout undo deploy/<pod名称>`                                    |
| **deployment回退pod到指定版本**                 | `kubectl rollout undo deploy/<pod名称> --to-revision=1`                    |
| **使用 kubectl edit 修改 Deployment 的镜像**    | `kubectl edit deployment my-deployment` (在编辑器中修改image字段)          |
| **使用 kubectl set 修改 Deployment 的镜像**     | `kubectl set image deployment/my-deployment my-container=nginx:1.19`       |
| **使用 kubectl replace 修改 Deployment 的镜像** | `kubectl replace -f my-deploy.yaml`                                        |

### 暴露服务
| 描述                                          | 命令                                                                            |
| :-------------------------------------------- | :------------------------------------------------------------------------------ |
| **deployment暴露pod集群内部访问 (ClusterIP)** | `kubectl expose deploy <pod名称> --port=8080 --target-port=80 --type=ClusterIP` |
| **deployment暴露pod外网访问 (NodePort)**      | `kubectl expose deploy <pod名称> --port=8080 --target-port=80 --type=NodePort`  |

## 5. Service 服务与网络
| 描述                                     | 命令                                |
| :--------------------------------------- | :---------------------------------- |
| **查看服务**                             | `kubectl get svc`                   |
| **查看服务详情**                         | `kubectl get svc -o wide`           |
| **查看所有名称空间下的服务**             | `kubectl get svc --all-namespaces`  |
| **查看特定名称空间下的 服务/pod/deploy** | `kubectl -n xxx get svc/pod/deploy` |
| **删除Service (svc)**                    | `kubectl delete svc xx`             |

## 6. 资源文件与高级操作
| 描述                                         | 命令                                                                |
| :------------------------------------------- | :------------------------------------------------------------------ |
| **使yaml文件生效**                           | `kubectl apply -f xxx.yaml`                                         |
| **或者一键删除一个yaml文件里定义的所有资源** | `kubectl delete -f XXX.yaml` (如果使用了PVC持久化存储先删除PV和PVC) |
| **删除当前文件夹下面所有yaml文件定义的资源** | `kubectl delete -f .`                                               |
| **无需删除pod等资源直接更新yaml内容**        | `kubectl apply -f xxx.yaml` 或 `kubectl replace -f xxx.yaml`        |
| **查看某个命名空间下的所有资源**             | `kubectl get all -n <namespace>`                                    |
| **查看configmap**                            | `kubectl get cm`                                                    |
| **重启对应deployment的pod**                  | `kubectl rollout restart deployment -n kube-system coredns` (示例)  |

> **⚠️ 注意事项**：
> * 先删 deploy 再删 pod，否则 pod 会自己创建。
> * pod 显示 Terminating 无法删除时，可能是使用了持久化存储，先删除对应 PVC 和 PV，还是不行则强制删除。