
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
