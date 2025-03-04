1. volumes
定义：volumes 是 Pod 级别的存储定义。它声明了 Pod 可以使用的存储资源类型和来源。

作用：volumes 定义了存储的来源（例如主机目录、持久化卷、配置映射等），但它并不直接指定存储挂载到容器的哪个路径。  

位置：volumes 在 Pod 的 spec 中定义    
例:    
```
volumes:
  - name: my-volume
    hostPath:
      path: /data
```
2.volumeMounts  
定义：volumeMounts 是容器级别的配置。它指定了将 volumes 挂载到容器内的哪个路径。  

作用：volumeMounts 将 volumes 中定义的存储挂载到容器的文件系统中，使容器可以访问这些存储。  

位置：volumeMounts 在容器的 spec 中定义。  
```
volumeMounts:
  - name: my-volume
    mountPath: /app/data
```
![image](https://github.com/user-attachments/assets/94b82bde-cb92-4f61-80a0-42738a51955d)  
volumes 和 volumeMounts 是配合使用的：  
 
volumes 定义了存储的来源。  

volumeMounts 将存储挂载到容器中。  

一个 volume 可以被多个容器挂载（通过 volumeMounts），但每个容器可以挂载到不同的路径。
