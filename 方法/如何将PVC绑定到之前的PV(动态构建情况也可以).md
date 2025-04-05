## PV有三种状态:Bound,Released,Available 
```
Bound:表示当前正在与某个PVC绑定
Released:表示与PVC解除了绑定，但是PV的 claimRef 仍指向旧 PVC，无法绑定到新 PVC
Available:PVC彻底释放，可以与新的PVC绑定
```

## 1 动态制备PV中，每创建一个PVC 制备器都会创建一个新的PV与其绑定 导致不能与旧PV绑定  
## 因此我们需要在PVC的yaml定义中指定旧的PV
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: jenkins-pvc
  namespace: kube-devops
spec:
  storageClassName: managed-nfs-storage
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
  volumeName: pvc-803fc786-b344-4ef0-aed3-4317cf6b48ad  # 这里新增，指定目标PV
```

## 2 接下来彻底释放PV的旧绑定  
这里我们之前绑定的PVC名字为：`jenkins-pvc` 在命名空间 `kube-devops ` 
旧的PV名字为:` pv-803fc786-b344-4ef0-aed3-4317cf6b48ad  `
```command  
# 1. 确保旧 PVC 已删除（如果存在）
kubectl delete pvc jenkins-pvc -n kube-devops --ignore-not-found

# 2. 清除 PV 的 claimRef 绑定
kubectl patch pv pv-803fc786-b344-4ef0-aed3-4317cf6b48ad -p '{"spec":{"claimRef":null}}'

# 3. 检查 PV 状态是否变为 Available
kubectl get pv pv-803fc786-b344-4ef0-aed3-4317cf6b48ad
```
## 3 应用指定旧的PV的yaml文件   
`kubectl apply -f xxx-pvc.yaml`  
查看  
`kubectl get pvc ..... `
`kubectl get pv  ..... `
