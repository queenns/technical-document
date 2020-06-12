# 安装

## 1.系统环境

```bash
$ cat /etc/redhat-release 
CentOS Linux release 7.5.1804 (Core)
```

## 2.Java安装

```bash
yum -y install java-1.8.0-openjdk
```

## 3.Jenkins安装

```bash
wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat/jenkins.repo
rpm --import https://pkg.jenkins.io/redhat/jenkins.io.key
yum upgrade
yum install jenkins
```

安装下载时速度太慢时，替换baseurl源地址

```bash
# cat /etc/yum.repos.d/jenkins.repo 
[jenkins]
name=Jenkins-stable
baseurl=https://mirrors.huaweicloud.com/jenkins/redhat-stable
gpgcheck=1
```

## 4.Jenkins启动/状态/停止

```bash
# systemctl start jenkins
# systemctl status jenkins
# systemctl stop jenkins
```

## 5. 安装启动完成

home目录: **/var/lib/jenkins** 使安装时默认创建的jenkins用户可登录

```bash
# cat /etc/passwd|grep jenkins
jenkins:x:995:994:Jenkins Automation Server:/var/lib/jenkins:/bin/false
# usermod -s /bin/bash jenkins
# cat /etc/passwd|grep jenkins
jenkins:x:995:994:Jenkins Automation Server:/var/lib/jenkins:/bin/bash
```

