# 软件版本
`Vmware`:16
`Debain`:13.3
`K8s`:1.34

# Debian 13.3 安装

## 1.前往官方或者镜像网站下载Debian 13.3 ISO镜像
 我下载的是：debian-13.3.0-amd64-DVD-1.iso

## 2.创建虚拟机并安装Debian 13.3
由于Vmware16版本较旧无法识别Debian 13.3镜像，所以创建虚拟机时选择较低的版本Debain即可，比如Debian 10。

安装过程参考B站视频[Debian13 保姆级安装教程](https://www.bilibili.com/video/BV1YEsazoEsS/?spm_id_from=333.337.search-card.all.click&vd_source=b9c639db66d1d92699bdd73eff797082)

**我这里安装时额外选择了镜像源站点中科大站点和SSH服务，方便后续操作，换源操作也可以安装之后在操作界面实现**

## 3.安装完成后进行基础配置
### **1.禁用DVD/ISO CD-ROM软件包源**

```
当你从DVD/ISO在计算机上安装Debian时，安装程序会配置cdrom软件包源。这个软件包源不会更新，并且在更新过程中会出现错误，因为该软件包源没有要更新的发行文件。

为确保你可以安装常规更新，请通过使用root命令访问/etc/apt/sources.list文件，并注释掉以deb cdrom开头的行。

使用你喜欢的Linux文本编辑器，比如Vim、vi或nano，运行以下命令：

nano /etc/apt/sources.list
```

### **2.(没有出现此问题)授予主用户超级用户权限**
用户账户可能没有超级用户访问权限。导致在非root 账户下输入密码也无法使用 sudo 命令。
![alt text](imgs/image.png)
```
出于安全原因，Debian 在安装期间不允许任何选项给予用户账户 sudo 访问权限。
要授予主用户 sudo 访问权限，请执行以下步骤：
```
**方法1**：
- 输入`su`使用 root 密码进入 root 账户
- 通过运行以下命令将本地用户添加为sudo用户：
`usermod -aG sudo <用户名>`
**方法2（没有usermod）**：
- 输入`su`使用root 密码进入 root 账户
- 编辑sudoers文件
`vi /etc/sudoers`
将`%sudo   ALL=(ALL:ALL) ALL`修改为`<你的用户名>   ALL=(ALL:ALL) ALL`
- 强制保存退出 `:wq!`
### **3.更新软件包列表并升级系统**
```
sudo apt update
sudo apt upgrade

```
### **4.安装常用工具**
`sudo apt install ffmpeg default-jdk git wget nano vim htop locate p7zip p7zip-full unzip`

### 5 配置clash 和安装docker(可选，有相关文档不做赘述) 


# K8s v1.34 安装
# 基础配置：
配置静态ip、修改主机名、配置hosts文件、重启生效、开启对应的端口、关闭swap分区、关闭防火墙和selinux(如果有)、配置时间同步等操作同之前版本一致
