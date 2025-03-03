![image](https://github.com/user-attachments/assets/6b731fee-a13c-457b-bc52-cf70caef575b)  
![image](https://github.com/user-attachments/assets/c489877b-a45b-4917-ab08-076410039289)

如果pod处于Terminating无法删除可以强制删除：  
强制删除：  
kubectl delete pod <pod_name> --force --grace-period=0

kubectl delete pod mysql-dep-74d9b967f6-rztck --force --grace-period=0

![image](https://github.com/user-attachments/assets/83224734-3255-4bc1-b5cd-af5518d869e3)
![image](https://github.com/user-attachments/assets/b95198c5-2c60-4cc4-b614-bc783d88af2a)

### /bin/sh 和 /bin/bash的区别：
/bin/sh 和 /bin/bash 都是 Unix/Linux 系统中的 Shell 解释器
sh 是 Bourne Shell 的缩写，是 Unix 系统中最经典的 Shell 解释器。
bash 是 Bourne-Again Shell 的缩写，是 sh 的增强版。
在容器中，/bin/sh 更常见，因为容器通常追求最小化，而 sh 的体积更小
果你希望脚本能在所有 Unix/Linux 系统上运行，应该使用 #!/bin/sh（即使用 sh 语法）。
如果你只需要在支持 bash 的系统上运行脚本，可以使用 #!/bin/bash
