## 问题：  
查看宿主机时间 `date`  

查看pod的日志，发现时间戳与宿主机不同步  


解决：  
这是因为原生k8s的默认pod时区为UTC(世界协调时间), 我们宿主机的为CST（中国标准时间)  
只需要把宿主机的时间文件usr/share/zoneinfo/Asia/Shanghai 挂载到 容器的/etc/localtime 即可    

pod的yaml定义:
```yaml
...
...
volumeMounts:
- name: timezone
mountPath: /etc/localtime # 挂载到容器的目录

volumes:
- name: timezone
hostPath:
path: /usr/share/zoneinfo/Asia/Shanghai # 宿主机的目录
```


