# Docker

## 1. 环境

```bash
# cat /etc/redhat-release 
CentOS Linux release 7.8.2003 (Core)
```

## 2. 安装

```bash
# yum-config-manager --add-repo http://download.docker.com/linux/centos/docker-ce.repo
# yum -y install docker-ce-18.03.1.ce
```

## 3. 启动/状态/停止

```bash
# systemctl start docker
# systemctl status docker
# systemctl stop doccker
```

