# 安装

## 1.机器准备
| HOSTNAME | IP | CPU | MEMORY |
|:--- |:--- |:--- |:--- |
|node66-103|10.20.66.103|4|8G|
|node66-104|10.20.66.104|4|8G|
|node66-105|10.20.66.105|4|8G|
|node66-106|10.20.66.106|4|8G|
|node66-107|10.20.66.107|4|8G|

## 2.设置主机名称
node66-103: 主机名称
host.com: 自定义域
```bash
[root@node66-103 ~]# hostnamectl set-hostname node66-103.host.com
[root@node66-103 ~]# hostnamectl set-hostname node66-104.host.com
[root@node66-103 ~]# hostnamectl set-hostname node66-105.host.com
[root@node66-103 ~]# hostnamectl set-hostname node66-106.host.com
[root@node66-103 ~]# hostnamectl set-hostname node66-107.host.com 
```

## 3.系统环境
```bash
[root@node66-103 ~]# cat /etc/redhat-release 
CentOS Linux release 7.5.1804 (Core) 
```