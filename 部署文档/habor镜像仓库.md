
# 1、下载安装包（Harbor 镜像使用 Docker-Compose 做编排）：

Harbor 下载链接：https://github.com/goharbor/harbor/releases ,   我这里使用的是 harbor-offline-installer-v2.12.2.tgz；
Docker-Compose 下载链接：https://github.com/docker/compose/releases,  我这里使用的是 docker-compose-linux-x86_64:v2.32.4；

# 2.[安装Docker](docker安装.md)

# 3.安装docker-compose  
由于这里我们下载的是 Docker-Compose 二进制包，所以直接添加执行权限就可以执行，
为方便后续使用这里我将它添加到 PATH环境变量包含的路径下（Path环境变量目录查看`echo $PATH`）  
`chmod +x docker-compose-linux-x86_64 && mv docker-compose-linux-x86_64 /usr/local/bin/docker-compose`
