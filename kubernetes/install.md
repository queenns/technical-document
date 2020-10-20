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
[~]# hostnamectl set-hostname node66-103.host.com
[~]# hostnamectl set-hostname node66-104.host.com
[~]# hostnamectl set-hostname node66-105.host.com
[~]# hostnamectl set-hostname node66-106.host.com
[~]# hostnamectl set-hostname node66-107.host.com 
```

## 3.系统环境
```bash
[~]# cat /etc/redhat-release 
CentOS Linux release 7.5.1804 (Core) 
[~]# uname -sr
Linux 3.10.0-862.el7.x86_64
```

## 4.关闭SElinux和firewalld
```bash
# 永久关闭SElinux需要重启,执行以下命令为临时关闭
[~]# setenforce 0
[~]# sed --follow-symlinks -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
[~]# getenforce 
Permissive

# 关闭防火墙
[~]# systemctl stop firewald
[~]# firewall-cmd --state
not running
```

## 5.安装epel
```bash
[~]# curl -o /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
[~]# yum install -y epel-release
```

## 6.安装必要工具根据需要确认是否修改Base源
```bash
[~]# curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo

[~]# yum install wget net-tools telnet tree nmap sysstat lrzsz dos2unix bind-utils -v
```

## 7.node66-103主机安装dns服务 bind9,更新dns配置
```bash
[root@node66-103 ~]# yum install -y bind
[root@node66-103 ~]# rpm -aq bind
bind-9.11.4-16.P2.el7_8.6.x86_64


[root@node66-103 ~]# vi /etc/named.conf
# listen-on port 53 { 10.20.66.103; };      ->    修改主机ip
# listen-on-v6 port 53 { ::1; };            ->    删除ipv6
# allow-query     { any; };                 ->    内网所有机器都允许查询
# forwarders      { 114.114.114.114; };     ->    上级dns
# recursion yes;                            ->    递归算法提供查询,另外一种是迭代查询
# dnssec-enable no;                         ->    dnssec功能关闭
# dnssec-validation no;                     ->    dnssec功能关闭

# 主机域 host.com
# 业务域 op.com
# 末尾添加以下内容,注意格式(空格,空行)
[root@node66-103 ~]# vi /etc/named.rfc1912.zones

zone "host.com" IN {
        type master;
        file "host.com.zone";
        allow-update { 10.20.66.103; };
};

zone "op.com" IN {
        type master;
        file "op.com.zone";
        allow-update { 10.20.66.103; };
};

[root@node66-103 ~]# vi /var/named/host.com.zone

$ORIGIN host.com.
$TTL 600        ; 10 minutes
@       IN SOA  dns.host.com. dnsadmin.host.com. (
                                2020101701 ; serial
                                10800      ; refresh (3 hours)
                                900        ; retry (15 minutes)
                                604800     ; expire (1 week)
                                86400      ; minimum (1 day)
                                )
                        NS   dns.host.com.
$TTL 60 ; 1 minute
dns               A    10.20.66.103
node66-103        A    10.20.66.103
node66-104        A    10.20.66.104
node66-105        A    10.20.66.105
node66-106        A    10.20.66.106
node66-107        A    10.20.66.107

[root@node66-103 ~]# vi /var/named/bs.com.zone

$ORIGIN op.com.
$TTL 600        ; 10 minutes
@       IN SOA  dns.op.com. dnsadmin.op.com. (
                                2020101701 ; serial
                                10800      ; refresh (3 hours)
                                900        ; retry (15 minutes)
                                604800     ; expire (1 week)
                                86400      ; minimum (1 day)
                                )
                        NS   dns.op.com.
$TTL 60 ; 1 minute
dns               A    10.20.66.103
node66-103        A    10.20.66.103
node66-104        A    10.20.66.104
node66-105        A    10.20.66.105
node66-106        A    10.20.66.106
node66-107        A    10.20.66.107

```
