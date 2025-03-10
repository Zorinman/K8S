[PV和PVC解释](https://github.com/Zorinman/K8S/blob/main/k8s%E5%9F%BA%E7%A1%80/%E6%9C%AF%E8%AF%AD%E8%A7%A3%E6%9E%90/PV%E5%92%8CPVC.md)  
⭐⭐⭐**以下内容对于namespace（命名空间的要求):**
# 命名空间关系总结

| 资源类型                         | 命名空间要求 |
|------------------------------|------------|
| **Nginx (Deployment)**       | 必须与 PVC 在同一命名空间（如 `default`） |
| **PVC**                      | 必须与应用（如 Nginx）在同一命名空间（如 `default`） |
| **Provisioner (Deployment)** | 固定为 `kube-system`（通常） |
| **StorageClass**             | 无命名空间 |
| **ServiceAccount(RBAC)**     | 必须与 Provisioner 在同一命名空间（如 `kube-system`） |
| **ClusterRole(RBAC)**              | 无命名空间 |
| **ClusterRoleBinding(RBAC)**       | 无命名空间，但需绑定到 Provisioner 的 ServiceAccount 所在命名空间 |
| **Role 和 RoleBinding(RBAC)**       | 必须与 Provisioner 在同一命名空间（如 `kube-system`） |



# 一、PV和PVC的动态构建（推荐）：
通过在PVC中设置StorageClass实现，如果集群中已经有的 PV 无法满足 PVC 的需求，那么集群会根据 PVC 自动构建一个 PV
每个 StorageClass 都有一个制备器（Provisioner），用来决定使用哪个卷插件制备 PV

整体过程：
### 1.[安装NFS创建共享目录](https://github.com/Zorinman/K8S/blob/main/%E9%83%A8%E7%BD%B2%E6%96%87%E6%A1%A3/NFS%E5%AE%89%E8%A3%85%E4%B8%8E%E9%85%8D%E7%BD%AE.md)  

### 2.创建Provisioner、StroageClass、PVC（PVC可以单独创建也可以在应用的模板里创建）  

### 3.在Provisioner中的volume字段指定使用的存储、在StroageClass中指定关联的Provisioner、在PVC的StroageClassName中指定关联的StroageClass  

### 4.创建RBAC配置  
作用是为 NFS Provisioner（动态存储卷分配器） 配置 Kubernetes RBAC 权限，使其能够动态创建和管理 PersistentVolume（PV），Provisioner 只需操作与存储相关的资源（PV/PVC/StorageClass），无需其他权限。
通过 ServiceAccount 和 RBAC 限制权限范围，避免越权操作）

### 5.创建应用,在应用的yaml文件中定义Volumes字段指向对应的PVC

### 6.制备器会根据PVC此时的状态来动态创建一个PV并绑定

-------------
**Provisioner模板：**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nfs-client-provisioner
  namespace: kube-system
  labels:
    app: nfs-client-provisioner
spec:
  replicas: 1
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: nfs-client-provisioner
  template:
    metadata:
      labels:
        app: nfs-client-provisioner
    spec:
      serviceAccountName: nfs-client-provisioner
      containers:
        - name: nfs-client-provisioner
          image: gcr.io/k8s-staging-sig-storage/nfs-subdir-external-provisioner:v4.0.0
          volumeMounts:
            - name: nfs-client-root
              mountPath: /persistentvolumes
          env:
            - name: PROVISIONER_NAME
              value: fuseim.pri/ifs   #环境变量：制备器的名字
            - name: NFS_SERVER
              value: 192.168.219.142   #环境变量：NFS服务器地址
            - name: NFS_PATH
              value: /nfs/data/01      #环境变量：NFS共享目录地址
      volumes:
        - name: nfs-client-root
          nfs:
            server: 192.168.219.142     #NFS服务器地址
            path: /nfs/data/01          #NFS共享目录地址
```

**StroageClass模板**：
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: managed-nfs-storage
  namespace: kube-system
provisioner: fuseim.pri/ifs # 必须与制备器的名字匹配
parameters:
  archiveOnDelete: "false" # 是否存档，false 表示不存档，会删除 oldPath 下面的数据，true 表示存档，会重命名路径
reclaimPolicy: Retain # 回收策略，默认为 Delete 可以配置为 Retain
volumeBindingMode: Immediate # 默认为 Immediate，表示创建 PVC 立即进行绑定，只有 azuredisk 和 AWSelasticblockstore 支持其他值
```

**PVC模板:**
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nginx-pvc
spec:
  storageClassName: managed-nfs-storage   #必须storageClass的名字相匹配
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi

```
**RBAC模板**
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nfs-client-provisioner
  namespace: kube-system
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: nfs-client-provisioner-runner
  namespace: kube-system
rules:
  - apiGroups: [""]
    resources: ["persistentvolumes"]
    verbs: ["get", "list", "watch", "create", "delete"]
  - apiGroups: [""]
    resources: ["persistentvolumeclaims"]
    verbs: ["get", "list", "watch", "update"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["storageclasses"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["events"]
    verbs: ["create", "update", "patch"]
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: run-nfs-client-provisioner
  namespace: kube-system
subjects:
  - kind: ServiceAccount
    name: nfs-client-provisioner
    namespace: kube-system
roleRef:
  kind: ClusterRole
  name: nfs-client-provisioner-runner
  apiGroup: rbac.authorization.k8s.io
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: leader-locking-nfs-client-provisioner
  namespace: kube-system
rules:
  - apiGroups: [""]
    resources: ["endpoints"]
    verbs: ["get", "list", "watch", "create", "update", "patch"]
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: leader-locking-nfs-client-provisioner
  namespace: kube-system
subjects:
  - kind: ServiceAccount
    name: nfs-client-provisioner	
    namespace: kube-system
roleRef:
  kind: Role
  name: leader-locking-nfs-client-provisioner
  apiGroup: rbac.authorization.k8s.io
```
**Nginx配置文件（测试应用）**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      volumes:
        - name: data-volume
          persistentVolumeClaim:
            claimName: nginx-pvc  #必须与PVC的名称一致
      containers:
        - name: nginx
          image: nginx
          volumeMounts:
            - name: data-volume
              mountPath: /usr/share/nginx/html
          ports:
            - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: 80
  selector:
    app: nginx
```

**创建**
```commandline
 创建 NFS Provisioner
kubectl apply -f nfs-provisioner.yaml
 创建 StorageClass
kubectl apply -f storageclass.yaml
 创建 PVC
kubectl apply -f pvc.yaml
 创建 rabc
kubectl apply -f rbac.yaml
 创建 Nginx 部署
kubectl apply -f nginx-deployment.yaml
```

# 二、PV和PVC的静态构建：
需要手动创建PV （不再需要Provisioner、StroageClass和RABC）
### 1.[安装NFS创建共享目录](https://github.com/Zorinman/K8S/blob/main/%E9%83%A8%E7%BD%B2%E6%96%87%E6%A1%A3/NFS%E5%AE%89%E8%A3%85%E4%B8%8E%E9%85%8D%E7%BD%AE.md) 
### 2.创建PV,PVC
### 3.创建应用,在应用的yaml文件中定义Volumes字段指向对应的PVC

**PVC模板:**
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nginx-pvc
spec:
  storageClassName: managed-nfs-storage   #必须PV中的storageClass的名字相匹配
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi

```

**PV模板**：
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv
spec:
  storageClassName: managed-nfs-storage  #必须PVC中的storageClass的名字相匹配
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  nfs:
    path: /nfs/data/01  #  NFS共享目录路径 
    server: 192.168.219.142  # NFS 服务器地址
```
**Nginx配置文件（测试应用）**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      volumes:
        - name: data-volume
          persistentVolumeClaim:
            claimName: nginx-pvc  #必须与PVC的名称一致
      containers:
        - name: nginx
          image: nginx
          volumeMounts:
            - name: data-volume
              mountPath: /usr/share/nginx/html
          ports:
            - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: 80
  selector:
    app: nginx
```
**创建**
```commandline
创建 PVC
kubectl apply -f pvc.yaml
创建PV
kubectl apply -f pvc.yaml
创建Nginx
kubectl apply -f nginx-deployment.yaml
```
