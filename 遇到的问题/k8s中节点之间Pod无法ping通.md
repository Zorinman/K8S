我的k8s使用的是calico网络插件
我发现我的k8s dashboard网站无法访问

## 1.使用命令查看Pod的ip以及其部署的节点
`kubectl get pods -A -o wide`  
发现dashboard的节点 以及calico网络插件都在正常运行
![image](https://github.com/user-attachments/assets/9c5b15ac-9ad0-4c7f-b57c-ffb1f4c3c8b4)
![image](https://github.com/user-attachments/assets/ff7f35d2-0543-4c49-99c4-21e8ece7fdd6)


## 2.查看service
`kubectl get svc -A`
Pod都显示在正常运行，尝试访问192.168.142:32393 发现无法打开dashboard的ui网站  
![image](https://github.com/user-attachments/assets/39607e36-5438-4902-918a-161b257c5f17)

## 3.尝试在master 主机上ping各个节点上的pod 发现均无法ping通  
![image](https://github.com/user-attachments/assets/ae1c6276-c527-4ef8-b5d0-bdd83c8b257f)

## 4.使用ifconfig命令查看网卡配置情况
`ifconfig`

发现没有tunl0网卡，因为默认calico的模式是IPIP，使用tunl0网卡在各个节点通信

## 5.在maser节点上重启网络配置

`systemctl stop NetworkManager`  
`systemctl disable NetworkManager`  
`service network restart`    
使用ifcong命令 发现tunI0网卡已经存在
![image](https://github.com/user-attachments/assets/6b88849c-5206-4588-88c6-3af3a60145a9)

## 6.继续尝试ping node1和node2上的pod  发现node1上的可以ping 通了 node2上的仍然没有反应

## 7.在node1 和node2上分别进行以上操作重启网络配置
`systemctl stop NetworkManager`  
`systemctl disable NetworkManager`  
`service network restart`  

## 8.之后继续尝试ping 发现各个节点的pod均能ping通
![image](https://github.com/user-attachments/assets/c30a7e7d-8760-4de4-9407-f1ba39339379)

## 9.之后尝试访问dashboard网站 成功访问
![image](https://github.com/user-attachments/assets/8d6adc5b-8b30-4650-a0c5-e5151f07ccc2)

解决了pod正常运行但无法在各个节点上互相ping通 以及k8s dashboard 无法访问的问题
