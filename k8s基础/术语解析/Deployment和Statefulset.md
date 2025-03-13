Deployment通常会在yaml的配置中通过配置persistentVolumeClaim为**多个Pod绑定一个创建好的PVC**
```
volumes:
      - name: my-storage
        persistentVolumeClaim:
          claimName: my-pvc
```
Statefulset通常通过volumeClaimTemplates**为多个Pod自动创建并绑定多个PVC**
```
volumeClaimTemplates:
  - metadata:
      name: mysql-data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 10Gi
```

![image](https://github.com/user-attachments/assets/fbae76fe-e79b-4240-b9d1-5c4dad6390f1)


![image](https://github.com/user-attachments/assets/c24a5e8c-696d-4465-9ccb-6d636b82b64b)
![image](https://github.com/user-attachments/assets/5024be4c-d7ef-489e-a2b8-b6e873dfd94d)
![image](https://github.com/user-attachments/assets/9cbfc541-69e1-4670-8bf7-5f255d0dcc0f)
