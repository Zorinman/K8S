**通常不建议同时使用 items 和 subPath**

**items** 是 ConfigMap 或 Secret 卷的一个字段（定义在volume中）
items 用于从 ConfigMap 或 Secret 中选择特定的键值对，并将它们挂载为文件
**使用场景**：
挂载多个文件，同时不需要避免容器内的其他文件被覆盖

------------------------------------------------------------------
**subPath** 是 定义在volumeMounts 中的一个字段
subPath 用于挂载卷中的单个文件或子目录，避免覆盖容器内的其他文件。
使用场景：
适合挂载单个文件或子目录同时避免容器中避免覆盖其他文件

如果需要挂载多个文件同时要避免容器文件覆盖
（使用subPath）：

```
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
    - name: my-container
      image: nginx
      volumeMounts:
        - name: config-volume
          mountPath: /app/config/key1-file  # 挂载 key1
          subPath: key1
        - name: config-volume
          mountPath: /app/config/key2-file  # 挂载 key2
          subPath: key2
  volumes:
    - name: config-volume
      configMap:
        name: my-config
```

总结：
将卷Volumes的Configmap中的key1和key2（文件或者键值都可）挂载到容器中的/app/config/key1-file
