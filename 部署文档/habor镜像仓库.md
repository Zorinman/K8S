
# 1、下载安装包（Harbor 镜像使用 Docker-Compose 做编排）：

Harbor 下载链接：https://github.com/goharbor/harbor/releases ,   我这里使用的是 harbor-offline-installer-v2.12.2.tgz；
Docker-Compose 下载链接：https://github.com/docker/compose/releases,  我这里使用的是 docker-compose-linux-x86_64:v2.32.4；

# 2.[安装Docker](docker安装.md)

# 3.安装docker-compose  
由于这里我们下载的是 Docker-Compose 二进制包，所以直接添加执行权限就可以执行，
为方便后续使用这里我将它添加到 PATH环境变量包含的路径下（Path环境变量目录查看`echo $PATH`）  
`chmod +x docker-compose-linux-x86_64 && mv docker-compose-linux-x86_64 /usr/local/bin/docker-compose`

# 4、配置 Harbor，因为 Harbor 是基于 Docker-Compose 启动的，所以我们下载是一个压缩包，其中包含一些 Harbor 所需镜像，如下  
```
# 解压 Harbor 压缩包 到opt目录
$ tar xf harbor-offline-installer-v2.12.2.tgz-C /opt

# 到解压后的harbor目录下去掉模板文件的后缀tmpl然后对模板进行编辑
$ cd /opt/harbor && mv harbor.yml.tmpl harbor.yml && vim harbor.yml

#habor我们一般在局域网用所以我们可以将https以下的内容注释掉，否则会需要SSL证书，另外可以将hostname的域名改为当前主机ip，harbor_admin_password登录密码，默认为Harbor12345,数据库database默认密码为root123  
#以上密码我都不做修改，Harbor的数据目录我修改为了/data/harbor，
#这里harbor的默认监听端口为80端口
#以下是模板文件里的部分关键内容：
# hostname: <Harbor 所在主机的 IP 或域名>
...
以下内容注释掉
#https:
    port:443
    certificate: /your/certificate/path
    private_key: /your/private/key/path
以上内容注释掉

#  harbor_admin_password: Harbor12345
...
#  data_volume: /data/harbor

#  
...
```
# 5.执行harbor安装脚本并查看是否正常启动
```
 $ ./install.sh 
 
 # 检查是否正常启动
$ docker-compose ps

```
# 6.浏览器访问hostname:80访问harbor管理端口  我这里是 192.168.219.129:80
 默认账户为admin
 默认密码为Harbor12345

# 7推送镜像至harbor仓库
适用场景：相同系统（这里是centos）下A主机镜像推到Harbor主机仓库，或者Harbor主机自己推送镜像到仓库
由于客户端docker默认采用的是Https协议，而我们在模板里使用的是http协议，  
因此要在**推送镜像的主机**上首先作如下**1.2**步的设置，否则会报错  

 **1.编辑docker配置文件**  
` vim /etc/docker/daemon.json`  
添加以下内容（这里是我自己的harbor主机ip）
```{
  "insecure-registries": ["192.168.219.129"]
}
```
 **2.重新加载、启动docker**
`systemctl daemon-reload
systemctl restart docker`

 如果在harbor主机上推送则重启docker后还要再次启动harbor
`docker-compose up -d`
------------------------------------------------------------
**接下来就可以开始正式推送镜像**

  **3.推送镜像主机的docker连接远程仓库**  
   `docker login 192.168.219.129`  
  （如果上述操作后连接时仍然报错可以尝试以下操作（①.②.③分别单独尝试）：  
 ①在 vim /etc/docker/daemon.json 中的ip地址后面加入镜像仓库的端口，如192.168.219.129:80  
 ②在docker login 192.168.219.129时加上端口如docker login 192.168.219.129:80  
 ③vim /etc/docker/daemon.json把里面添加的ip内容清空，改为在docker.service文件中配置

  ```
  #查找docker.service所在位置 
  $find / -name docker.service -type f
  
  #find: ‘/run/user/1000/gvfs’: Permission denied
/usr/lib/systemd/system/docker.service
  #编辑配置文件 
  $vim /usr/lib/systemd/system/docker.service
  #找到ExecStart= ，并将以下内容添加到后面
  --insecure-registry=192.168.219.129
  #保存退出
  #重启docker服务
  $systemctl daemon-reload
  $systemctl restart docker
  ``` 
）

  **4.给待推送的镜像打标记，打标记命令格式如下：**  
  `docker tag SOURCE_IMAGE[:TAG] 192.168.219.129/library/REPOSITORY[:TAG]`该命令可在harbor仓库查看  
  SOURCE_IMAGE[:TAG]表示当前docker已存在的某个版本的镜像名称  
  library表示的是harbor里头的某个项目名称，表示镜像推送给这个项目  
  REPOSITORY[:TAG]表示的是推送到仓库后你想以什么名称保存,可以随意设置  
  **例：**  
  如果我要推送一个 REPOSITORY 为mysql,TAG为5.7的镜像  
  则可以为`docker tag mysql:5.7 192.168.219.129/library/This is mysql:5.7`
    
 **5.推送完成后断开连接**
 `docker logout`
 
# 8.从仓库拉取镜像
适用场景同推送镜像
1.2.3步同推送镜像，

**4.在浏览器登录harbor然后在harbor仓库选择要拉取的镜像，直接复制拉取镜像的命令即可**
![image](https://github.com/user-attachments/assets/39fd6107-ad60-4eaf-8fc8-55c634f97b00)


**5.粘贴命令至需要拉取镜像的主机即可开始拉取**

# 9.关于harbor的补充
如果以后需要修改harbor.yml文件，那么先停止harbor`docker-compose down`，再去修改harbor.yml文件，然后重新启动部署 `./install.sh`

正常启动harbor启动：`docker-compose up -d`

未修改配置文件，重启Harbor命令：`docker-compose start | stop | restart`
————————————————

