使用kube-prometheus安装的grafana，安装之后能正常使用，在我部署了ELK之后由于节点没有给足够的内存导致虚拟机崩溃，等解决崩溃虚拟机重启后，发现grafana不能正常访问网站  

查看grafana的pod中的容器日志 发现报错："database is locked"  
`kubectl logs <pod-name> -n <namespace-name> -c grafana`   

(grafana的容器名通常为grafana，也可在资源文件里查看)

### 原因：
 由于节点崩溃后Grafana的数据库文件grafana.db出现数据状态不一致导致文件被锁定，导致grafana无法正常访问 SQLite 数据库并无法正常启动

### 解决：
**1.首先查看Grafana的资源配置文件，看其grafana.db文件在哪个位置**  
以yaml文件信息输出grafana的deployment的配置信息  

`kubectl get deployment grafana  -n monitoring  -oyaml`  

查找关键词grafana-storage  

看到以下信息
```yaml
 volumeMounts:
        - mountPath: /var/lib/grafana
          name: grafana-storage
 volumes:
      - emptyDir: {}
        name: grafana-storage
```
这表明grafana.db是以容器临时存储保存的，该卷挂载在容器内的/var/lib/grafana目录，
**临时存储的情况下可以试着直接重启deployment看是否能够解决**

`kubectl rollout restart -n <namespcae-name> deployment <deployment-name>`  

**2.将容器内的grafana.db文件拉取到本地并生成新的未被锁定文件**

`kubectl cp -n <namespcae-name> <grafana_pod_name>:/var/lib/grafana/grafana.db -c grafana ./grafana.db`

复制文件内数据生成新的grafana-new.db文件，把被锁定的旧文件改名为grafana-old.db，新的grafana-new.db改名为grafana.db

```commandline
$ sqlite3 grafana.db ".backup grafana-new.db"
$ mv grafana.db grafana-old.db
$ mv grafana-new.db grafana.db
```

**3.进入容器删除被锁定的grafana.db文件并将新的grafana.db移动到容器内**
`kubectl exec -n <namespace-name>  -it <pod-name> -c grafana -- sh`  

在容器内执行命令：
```commandline
$ cd /var/lib/grafana
$ rm grafana.db
#退出容器
```
在本地未被锁定的grafana.db存放位置执行命令，将新的grafana.db移动到容器内对应位置
`kubectl cp ./grafana.db -n <namespcae-name> <grafana_pod_name>:/var/lib/grafana/grafana.db -c grafana`

**4.重启grafana的deployment使得更改信息被更新**

`kubectl rollout restart -n <namespcae-name> deployment <deployment-name>`   

查看pod更新情况，完成后即可成功访问grafana服务

之后就可以把本地grafana-old.db和grafana.db文件删除了

-------------------

## **第二种情况**
```yaml
volumes:
      - emptyDir: {}
        name: grafana-storage
```


如果此处Volumes使用的不是临时存储，而是PVC，PV持久存储，那么只需要找到本地持久化存储的目录然后执行  
```commandline
$ sqlite3 grafana.db ".backup grafana-new.db"
$ mv grafana.db grafana-old.db
$ mv grafana-new.db grafana.db
```
**重启grafana的deployment使得更改信息被更新**
`kubectl rollout restart deployment <deployment-name>`  

然后就可以删除本地持久存储目录下的grafana-old.db文件
