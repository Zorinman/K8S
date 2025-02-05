# Docker安装时常常会碰见一系列的网络问题 如curl wget 命令获取失败，按照以下方法可以解决

## 一.ubuntu下采用一键安装：https://fishros.org.cn/forum/topic/20/%E5%B0%8F%E9%B1%BC%E7%9A%84%E4%B8%80%E9%94%AE%E5%AE%89%E8%A3%85%E7%B3%BB%E5%88%97

## 二.centos下因为没有apt和python3 所以上面一键安装的脚本用不了，改用以下方法：

### 1.更换为阿里源：

将docker官方镜像源更换为阿里云的源，将 Docker 的阿里云镜像源添加到 CentOS 系统中的一个仓库：
 `yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo `  

## 2.查看docker仓库中的版本并选择要安装的docker版本：
`yum list docker-ce --showduplicates | sort -r `  
我们可以安装指定版本，使用  `sudo yum install <docker的版本> `   
使用 `sudo yum install docker-ce`  默认安装最高的版本。  

## 3.启动docker并设置为开机启动：
`sudo systemctl start docker ` 
`sudo systemctl enable docker`

## 4.检测是否安装docker成功：
`docker -v` 查看docker版本
