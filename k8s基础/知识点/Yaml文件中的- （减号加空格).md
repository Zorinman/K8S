在 YAML 文件中，-（减号加空格） 用于表示列表项（数组或列表）。它的使用场景主要是在定义数组或列表时，每个列表项前需要加上 -  
```
1. 使用 - 的场景
场景 1：定义数组或列表
当某个字段的值是一个数组（列表）时，每个数组元素前需要加 -。

示例：

yaml
fruits:
  - apple
  - banana
  - orange
#这里的 fruits 是一个数组，包含三个元素：apple、banana 和 orange。

#每个元素前都需要加 -。
```
```
2.Kubernetes 中的列表
#在 Kubernetes 的 YAML 文件中，- 常用于定义多个容器、卷、环境变量等
containers:
  - name: nginx
    image: nginx:1.7.9
  - name: redis
    image: redis:latest
#这里的 containers 是一个列表，包含两个容器：nginx 和 redis。

每个容器定义前都需要加 -
```
```
Kubernetes 中的完整用法
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
    - name: nginx
      image: nginx:1.7.9
    - name: redis
      image: redis:latest
  volumes:
    - name: data
      emptyDir: {}
containers 和 volumes 都是列表，每个元素前都加了 -。
```
