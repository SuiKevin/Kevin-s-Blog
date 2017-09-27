
# Ambari安装Hadoop集群教程

## 集群环境：

### 阿里云服务器

#### 系统：

Centos7

Linux master 3.10.0-514.16.1.el7.x86_64 #1 SMP Wed Apr 12 15:04:24 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux

#### 配置：

master * 1 -> CPU : 4 core RAM : 16G OS_DISK : 40G DATA_DISK : 1T

slave * 2 -> CPU : 4 core RAM : 8G OS_DISK : 40G DATA_DISK : 1T

## 版本

### Ambari

Ambari-2.5.2.0-centos7

### HDP

HDP-2.6.1.0-centos7

### HDP-UTILS

HDP-UTILS-1.1.0.21-centos7


# 准备环境

## pyenv installation
使用pyenv创建虚拟python环境，可用来装anaconda环境，供pyspark程序使用
```
#安装依赖
yum install readline readline-devel readline-static -y
yum install openssl openssl-devel openssl-static -y
yum install sqlite-devel -y
yum install bzip2-devel bzip2-libs -y
yum install git-core bzip2 -y

#下载并执行自动安装脚本
curl -L https://raw.github.com/yyuu/pyenv-installer/master/bin/pyenv-installer | bash

#添加下面代码到~/.bashrc
export PATH="/root/.pyenv/bin:$PATH"
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"

#下载python安装包
wget http://mirrors.sohu.com/python/3.6.2/Python-3.6.2.tar.xz

#创建pyenv下载缓存目录
mkdir ~/.pyenv/cache

#将已下载的python安装包复制到pyenv下载缓存目录
cp Python-3.6.2.tar.xz ~/.pyenv/cache

#pyenv安装python，如果cache目录有对应版本安装包，就使用其进行安装，否则会在线下载安装包（墙内慢到死）
pyenv install 3.6.2

#设置全局系统python版本（hadoop生态相关组件可能不支持python3）
#pyenv global 3.6.2
```

## nginx installation
使用nginx作为ambari离线仓库，进行ambari、HDP、HDP-Utils安装
```
#安装nginx
yum install nginx -y

#编辑nginx配置文件
vim /etc/nginx/nginx.conf
#修改80端口的server中的root目录为离线安装包的目录
root  /mnt/ambari;

#测试nginx配置文件是否正确
nginx -t

#启动nginx
nginx
```

## 配置集群免密登录
```
#创建ssh密钥对
ssh-keygen

#输出公钥到authorized_keys文件
cat id_rsa.pub >> authorized_keys

#在集群中所有的服务器中创建.ssh目录
mkdir ~/.ssh

#复制authorized_keys文件到集群所有机器上的.ssh目录中
#修改对应目录及文件权限
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

## 配置DNS
```
vim /etc/hosts
#在文件中添加集群所有机器的IP和FQDN，例如：
10.30.xxx.xx1 master
10.31.xxx.xx2 slave1
10.31.xxx.xx3 slave2
```

## ntp installation
```
yum install -y ntp
chkconfig ntpd on
```

## set hostname
```
hostname master
#hostname slave1
#hostname slave2

vim /etc/sysconfig/network
#添加/修改HOSTNAME
HOSTNAME=master
#HOSTNAME=slave1
#HOSTNAME=slave2
```

## 关闭防火墙（CENTOS 7）
```
chkconfig firewalld off
service firewalld stop
```

## 禁用selinux
```
#查看selinux状态
sestatus

#禁用selinux
vim /etc/yum/pluginconf.d/refresh-packagekit.conf
setenforce 0
```

## 检查umask值
```
#检查umask值
umask

#设置umask值
echo umask 0022 >> /etc/profile
```

## 开启PackageKit
```
/etc/yum/pluginconf.d/refresh-packagekit.conf
enabled=0
```


# 安装ambari

## 添加本地仓库
**Ambari Base URL**
```
http://<web.server>/Ambari-2.5.1.0/<OS>
http://master/ambari/centos7
```
**HDF Base URL**
```
http://<web.server>/hdf/HDF/<OS>/3.x/updates/<latest.version>
http://<web.server>/hdp/HDP/<OS>/2.x/updates/<latest.version>
http://master/HDP/centos7
```
**HDP-UTILS Base URL**
```
http://<web.server>/hdp/HDP-UTILS-<version>/repos/<OS>
http://master/HDP-UTILS-1.1.0.21
```

**创建ambari.repo文件**
```
vim /etc/yum.repos.d/ambari.repo

[Updates-Ambari-2.5.2.0]
name=Ambari-2.5.2.0-Updates
baseurl=http://10.30.xxx.xx1/ambari/centos7
gpgcheck=1
gpgkey=http://10.30.xxx.xx1/ambari/centos7/RPM-GPG-KEY/RPM-GPG-KEY-Jenkins
enabled=1
priority=1
```

**Optional: If you have multiple repositories configured in your environment, deploy the following plug-in on all the nodes in your cluster.**

a.	Install the plug-in.
```
yum install yum-plugin-priorities
```

b.	Edit the /etc/yum/pluginconf.d/priorities.conf file to add the following:
```
[main]
enabled=1
gpgcheck=0
```

## 下载离线安装包
```
#ambari离线安装包
wget http://public-repo-1.hortonworks.com/ambari/centos7/2.x/updates/2.5.1.0/ambari-2.5.1.0-centos7.tar.gz

#HDP安装包版本信息
wget http://public-repo-1.hortonworks.com/HDP/centos7/2.x/updates/2.6.1.0/HDP-2.6.1.0-129.xml
#HDP离线安装包
wget http://public-repo-1.hortonworks.com/HDP/centos7/2.x/updates/2.6.1.0/HDP-2.6.1.0-centos7-rpm.tar.gz

#HDP-UTILS离线安装包
wget http://public-repo-1.hortonworks.com/HDP-UTILS-1.1.0.21/repos/centos7/HDP-UTILS-1.1.0.21-centos7.tar.gz
```

## 安装ambari-server
```
yum install ambari-server -y
```

## 配置ambari-server
```
ambari-server setup
```
**如果jdk安装过慢，先下载好jdk安装包再安装ambari-server**
```
#ambari原始jdk下载地址
#wget http://public-repo-1.hortonworks.com/ARTIFACTS/jdk-8u112-linux-x64.tar.gz -O /var/lib/ambari-server/resources/jdk-8u112-linux-x64.tar.gz

#oracle官网jdk下载地址
#wget --no-cookies \
#--no-check-certificate \
#--header "Cookie: oraclelicense=accept-securebackup-cookie" \
#"http://public-repo-1.hortonworks.com/ARTIFACTS/jdk-8u112-linux-x64.tar.gz" \
#-O /var/lib/ambari-server/resources/jdk-8u112-linux-x64.tar.gz

#国内jdk镜像下载地址[推荐]
wget http://mirrors.linuxeye.com/jdk/jdk-8u112-linux-x64.tar.gz -O /var/lib/ambari-server/resources/jdk-8u112-linux-x64.tar.gz
```

## 启动ambari-server
```
ambari-server start
```

## ambari访问地址
http://master:8080


# 建立集群

## add version
http://public-repo-1.hortonworks.com/HDP/centos7/2.x/updates/2.6.1.0/HDP-2.6.1.0-129.xml

## add cluster
**cluster FQDNs**
```
master
slave1
slave2
```

## 添加私钥文件
**id_rsa**


# Error1

```
# slave1: parent directory /usr/hdp/current/hadoop-client/conf doesn't exist

创建对应目录
```


# Note1

```
# read file from hdfs url

hdfs://+path

# example: hdfs:///tmp/ProjectFile/*.dat

# note: // is protocol, / is root
```
