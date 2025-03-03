![image](https://github.com/user-attachments/assets/6b731fee-a13c-457b-bc52-cf70caef575b)  
![image](https://github.com/user-attachments/assets/2941f703-7156-4d0e-9f24-34bf6a1d6d62)  
如果pod处于Terminating无法删除可以强制删除：  
强制删除：  
kubectl delete pod <pod_name> --force --grace-period=0

kubectl delete pod mysql-dep-74d9b967f6-rztck --force --grace-period=0

![image](https://github.com/user-attachments/assets/83224734-3255-4bc1-b5cd-af5518d869e3)
![image](https://github.com/user-attachments/assets/b95198c5-2c60-4cc4-b614-bc783d88af2a)





