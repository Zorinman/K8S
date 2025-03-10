PV 和 PVC 的设计简化了存储管理，实现了存储解耦，PV可以与多种存储如磁盘、NFS、CRI等集成，并与PVC进行绑定，PVC 通过抽象层屏蔽了底层差异，Pod只需要在YAML中定义PVC就能获取想要的存储资源    
```
PV中定义存储的空间，存储的类型配置以及与PVC关联的模型（一对一，一对多）等信息  
PVC中定义了需要的存储空间和存储类型等。  
当Pod定义PVC后，PVC就会根据自己的配置信息找到对应存储大小对应存储类型的PV进行绑定。  
```
```
持久卷（PersistentVolume，PV） 是集群中的一块存储，可以由管理员事先制备，
或者使用存储类（Storage Class）来动态制备。 持久卷是集群资源，就像节点也是集群资源一样。PV 持久卷和普通的 Volume 一样，
也是使用卷插件来实现的，只是它们拥有独立于任何使用 PV 的 Pod 的生命周期。
此 API 对象中记述了存储的实现细节，无论其背后是 磁盘、NFS、iSCSI 还是特定于云平台的存储系统。
```

```
持久卷申领（PersistentVolumeClaim，PVC） 表达的是用户对存储的请求。概念上与 Pod 类似。
Pod 会耗用节点资源，而 PVC 申领会耗用 PV 资源。Pod 可以请求特定数量的资源（CPU 和内存）；
同样 PVC 申领也可以请求特定的大小和访问模式 
（例如，可以要求 PV 卷能够以 ReadWriteOnce、ReadOnlyMany 或 ReadWriteMany 模式之一来挂载，参见访问模式）
```
![img_6](https://github.com/user-attachments/assets/8a795bbe-ada1-41c0-ad26-44059ff78c4f)

**流程：**  
提前设置好存储并且手动创建好PV，PVC
在Pod的Yaml文件中的Volume字段定义所需PVC，PVC会根据存储大小以及存储类型自动找到PV进行绑定，PV关联了对应的存储，这样Pod就能够进行持久化存储  
一个PV可以与一个或者多个PVC绑定

### **PV和PVC分为静态创建和动态创建**

**静态创建**：需要集群管理员手动创建若干 PV 卷。**（如上）**

**动态创建（推荐）**：  
通过在PVC中设置StorageClass实现，如果集群中已经有的 PV 无法满足 PVC 的需求，那么集群会根据 PVC 自动构建一个 PV  
每个 StorageClass 都有一个制备器（Provisioner），用来决定使用哪个卷插件制备 PV
![img_7](https://github.com/user-attachments/assets/eaaa3d15-f2b8-49f0-85d5-6de47fc50b7b)


静态创建和动态创建PV和PVC并应用到Pod上的示例：
