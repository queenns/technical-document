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

[root@node66-103 ~]# vi /var/named/op.com.zone

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
harbor            A    10.20.66.103

[root@node66-103 ~]# named-checkconf
[root@node66-103 ~]# systemctl restart named
[root@node66-103 ~]# netstat -luntp | grep 53

[root@node66-103 ~]# dig -t A node66-103.host.com @10.20.66.103 +short
10.20.66.103
[root@node66-103 ~]# dig -t A node66-104.host.com @10.20.66.103 +short
10.20.66.104
[root@node66-103 ~]# dig -t A node66-105.host.com @10.20.66.103 +short
10.20.66.105
[root@node66-103 ~]# dig -t A node66-106.host.com @10.20.66.103 +short
10.20.66.106
[root@node66-103 ~]# dig -t A node66-107.host.com @10.20.66.103 +short
10.20.66.107

# update 104-107 dns 10.20.66.103
[~]vi /etc/sysconfig/network-scripts/ifcfg-eth0
[~]systemctl restart network
```

## 证书
```sh
[~]# wget https://pkg.cfssl.org/R1.2/cfssl_linux-amd64 -O /usr/bin/cfssl
[~]# wget https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64 -O /usr/bin/cfssl-json
[~]# wget https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64 -O  /usr/bin/cfssl-certinfo
[~]# chmod +x /usr/bin/cfssl*

[~]# mkdir -p /opt/certs
[~]# cd /opt/certs

# CN: Common Name,浏览器使用该字段验证网站是否合法,一般写的是域名,非常重要
# C: Country 国家
# ST: State 州,省
# L: Locality 地区,城市
# O: Organization Name 组织名称,公司名称　
# OU: Organization Unit Name 组织单位名称,公司部门
[~]# vi /opt/certs/ca-csr.json
{
  "CN": "LiuxiaojianDebug",
  "hosts": [],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "beijing",
      "L": "beijing",
      "O": "ONLXJ",
      "OU": "ONLXJTD"
    }
  ],
  "ca": {
    "expiry": "175200h"
  }
}
[~]# cfssl gencert -initca ca-csr.json | cfssl-json -bare ca
```

## Docker
```sh
[~]# curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
[~]# mkdir -p /etc/docker /var/lib/docker
[~]# vi /etc/docker/daemon.json
{
  "graph": "/var/lib/docker",
  "storage-driver": "overlay2",
  "insecure-registries": ["harbor.op.com"],
  "bip": "172.1.1.1/24",
  "exec-opts": ["native.cgroupdriver=systemd"],
  "live-restore": true
}
[~]# systemctl start docker
```

## Harbor
```sh
[~]# mkdir -p /opt/src
[/opt/src]# wget https://github.com/goharbor/harbor/releases/download/v1.8.5/harbor-offline-installer-v1.8.5.tgz
[~]# tar -xvf harbor-offline-installer-v1.8.5.tgz -C /opt
[/opt]# mv harbor harbor-v1.8.5
[/opt]# ln -s /opt/harbor-v1.8.5 /opt/harbor

[/opt]# yum install -y docker-compose
[/opt/harbor]# vi harbor/harbor.yml
hostname: harbor.op.com
http:
  port: 180
[/opt/harbor]# sh install.sh
[/opt/harbor]# docker-compose ps
Name                     Command               State             Ports          
--------------------------------------------------------------------------------------
harbor-core         /harbor/start.sh                 Up                               
harbor-db           /entrypoint.sh postgres          Up      5432/tcp                 
harbor-jobservice   /harbor/start.sh                 Up                               
harbor-log          /bin/sh -c /usr/local/bin/ ...   Up      127.0.0.1:1514->10514/tcp
harbor-portal       nginx -g daemon off;             Up      80/tcp                   
nginx               nginx -g daemon off;             Up      0.0.0.0:180->80/tcp      
redis               docker-entrypoint.sh redis ...   Up      6379/tcp                 
registry            /entrypoint.sh /etc/regist ...   Up      5000/tcp                 
registryctl         /harbor/start.sh                 Up
[~]# yum install -y nginx
[~]# vi /etc/nginx/conf.d/harbor.op.com.conf
server{
    listen                 80;
    server_name            harbor.op.com;
    client_max_body_size   1024m;
    location / {
        proxy_pass http://127.0.0.1:180;
    }
}
[~]# nginx -t
[~]# systemctl start nginx
[~]# systemctl enable nginx

# 验证每个节点对harbor.op.com是否有解析
[~]# dig -t A harbor.op.com +short
10.20.66.103
```

## Etcd
```sh
[~]# vi /opt/certs/ca-config.json
{
    "signing": {
        "default": {
            "expiry": "175200h"
        },
        "profiles": {
            "server": {
                "expiry": "175200h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "server auth"
                ]
            },
            "client": {
                "expiry": "175200h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "client auth"
                ]
            },
            "peer": {
                "expiry": "175200h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "server auth",
                    "client auth"
                ]
            }
        }
    }
}
[~]# vi /opt/certs/etcd-peer-csr.json
{
    "CN": "kubernetes-etcd",
    "hosts": [
        "10.20.66.105",
        "10.20.66.106",
        "10.20.66.107"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [{
        "C": "CN",
        "ST": "beijing",
        "L": "beijing",
        "O": "ONLXJ",
        "OU": "ONLXJTD"
    }]
}
[/opt/certs]# cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=peer etcd-peer-csr.json | cfssl-json -bare etcd-peer
[/opt/certs]# ls -al
总用量 44
drwxr-xr-x. 2 root root 4096 10月 23 19:51 .
drwxr-xr-x. 6 root root 4096 10月 22 23:04 ..
-rw-r--r--  1 root root  836 10月 23 19:45 ca-config.json
-rw-r--r--. 1 root root 1013 10月 22 21:52 ca.csr
-rw-r--r--. 1 root root  271 10月 22 21:51 ca-csr.json
-rw-------. 1 root root 1675 10月 22 21:52 ca-key.pem
-rw-r--r--. 1 root root 1383 10月 22 21:52 ca.pem
-rw-r--r--  1 root root 1074 10月 23 19:51 etcd-peer.csr
-rw-r--r--  1 root root  327 10月 23 19:50 etcd-peer-csr.json
-rw-------  1 root root 1675 10月 23 19:51 etcd-peer-key.pem
-rw-r--r--  1 root root 1456 10月 23 19:51 etcd-peer.pem

[~]# useradd -s /sbin/nologin -M etcd
[~]#  wget https://github.com/etcd-io/etcd/releases/download/v3.1.20/etcd-v3.1.20-linux-amd64.tar.gz
[~]# tar -xvf etcd-v3.1.20-linux-amd64.tar.gz -C /opt
[~]# mv /opt/etcd-v3.1.20-linux-amd64 /opt/etcd-v3.1.20
[~]# ln -s /opt/etcd-v3.1.20 /opt/etcd

[~]# mkdir -p /opt/etcd/certs /export/etcd/etcd-server /export/etcd/logs
[~]# scp node66-103:/opt/certs/ca.pem /opt/etcd/certs
[~]# scp node66-103:/opt/certs/etcd-peer.pem /opt/etcd/certs
[~]# scp node66-103:/opt/certs/etcd-peer-key.pem /opt/etcd/certs

[~]# vi /opt/etcd/etcd-server-startup.sh

#!/bin/sh
./etcd --name etcd-server-66-105 \
       --data-dir /export/etcd/etcd-server \
       --listen-peer-urls https://10.20.66.105:2380 \
       --listen-client-urls https://10.20.66.105:2379,http://127.0.0.1:2379 \
       --quota-backend-bytes 8000000000 \
       --initial-advertise-peer-urls https://10.20.66.105:2380 \
       --advertise-client-urls https://10.20.66.105:2379,http://127.0.0.1:2379 \
       --initial-cluster etcd-server-66-105=https://10.20.66.105:2380,etcd-server-66-106=https://10.20.66.106:2380,etcd-server-66-107=https://10.20.66.107:2380 \
       --ca-file ./certs/ca.pem \
       --cert-file ./certs/etcd-peer.pem \
       --key-file ./certs/etcd-peer-key.pem \
       --client-cert-auth \
       --trusted-ca-file ./certs/ca.pem \
       --peer-ca-file ./certs/ca.pem \
       --peer-cert-file ./certs/etcd-peer.pem \
       --peer-key-file ./certs/etcd-peer-key.pem \
       --peer-client-cert-auth \
       --peer-trusted-ca-file ./certs/ca.pem \
       --log-output stdout
[~]# chmod +x /opt/etcd/etcd-server-startup.sh
[~]# chown -R etcd:etcd /opt/etcd-v3.1.20
[~]# chown -R etcd:etcd /export/etcd

[~]# yum install -y supervisor
[~]# systemctl start supervisord
[~]# systemctl enable supervisord

[~]# vi /etc/supervisord.d/etcd-server.ini

[program:etcd-server-66-105]
command=/opt/etcd/etcd-server-startup.sh           ; the program(relative uses PATH, can take args)
numprocs=1                                         ; number of processes copies to start (def 1)
directory=/opt/etcd                                ; directory to cwd to before exec (def no cwd)
autostart=true                                     ; start at supervisord start (default: true)
autorestar=true                                    ; restart at unexpected quit (default: true)
startsecs=30                                       ; number of secs prog must stay running (def. 1)
startretries=3                                     ; max # of serial start failures (default 3)
exitcodes=0,2                                      ; 'expected' exit codes for process (default 0,2)
stopsignal=QUIT                                    ; signal used to kill process (default TERM)
stopwaitsecs=10                                    ; max num secs to wait b4 SIGKILL (default 10)
user=etcd                                          ; setuid to this UNIX account to run the program
redirect_stderr=true                               ; redirect proc stderr to stdout (default false)
stdout_logfile=/export/etcd/logs/etcd.stdout.log   ; stdout log path, NONE for none; default AUTO
stdout_logfile_maxbytes=64MB                       ; max # logfile bytes b4 rotation (default 50MB)
stdout_logfile_backups=4                           ; # of stdout logfile backups (default 10)
stdout_capture_maxbytes=1MB                        ; number of bytes in 'capturemode' (default 0)
stdout_events_enable=false                         ; emit events on stdout writes (default false)

[~]# supervisorctl update
etcd-server-66-105: added process group
[~]# supervisorctl status
etcd-server-66-105               RUNNING   pid 22288, uptime 0:00:38
[~]# netstat -luntp | grep etcd
tcp        0      0 10.20.66.105:2379       0.0.0.0:*               LISTEN      22289/./etcd        
tcp        0      0 127.0.0.1:2379          0.0.0.0:*               LISTEN      22289/./etcd        
tcp        0      0 10.20.66.105:2380       0.0.0.0:*               LISTEN      22289/./etcd

[~]# /opt/etcd/etcdctl cluster-health
member 3dd3e7964e370c04 is healthy: got healthy result from http://127.0.0.1:2379
member acf5c0b7dea7bcf4 is healthy: got healthy result from http://127.0.0.1:2379
member c21bf6bec3e41b76 is healthy: got healthy result from http://127.0.0.1:2379
cluster is healthy

[~]# /opt/etcd/etcdctl member list
3dd3e7964e370c04: name=etcd-server-66-105 peerURLs=https://10.20.66.105:2380 clientURLs=http://127.0.0.1:2379,https://10.20.66.105:2379 isLeader=true
acf5c0b7dea7bcf4: name=etcd-server-66-106 peerURLs=https://10.20.66.106:2380 clientURLs=http://127.0.0.1:2379,https://10.20.66.106:2379 isLeader=false
c21bf6bec3e41b76: name=etcd-server-66-107 peerURLs=https://10.20.66.107:2380 clientURLs=http://127.0.0.1:2379,https://10.20.66.107:2379 isLeader=false

```

## apiserver
```sh
[~]# wget https://dl.k8s.io/v1.16.15/kubernetes-server-linux-amd64.tar.gz

[~]# vi client-csr.json

{
    "CN": "kubernetes-node",
    "hosts": [],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [{
        "C": "CN",
        "ST": "beijing",
        "L": "beijing",
        "O": "ONLXJ",
        "OU": "ONLXJTD"
    }]
}

[~]# cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=client client-csr.json | cfssl-json -bare client

[~]# vi apiserver-csr.json

{
    "CN": "kubernetes-apiserver",
    "hosts": [
        "127.0.0.1",
        "kubernetes.default",
        "kubernetes.default.svc",
        "kubernetes.default.svc.cluster",
        "kubernetes.default.svc.cluster.local",
        "10.20.66.103",
        "10.20.66.104",
        "10.20.66.105",
        "10.20.66.106",
        "10.20.66.107"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [{
        "C": "CN",
        "ST": "beijing",
        "L": "beijing",
        "O": "ONLXJ",
        "OU": "ONLXJTD"
    }]
}

[~]# cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=server apiserver-csr.json | cfssl-json -bare apiserver

[/opt/src]# tar -xvf kubernetes-server-linux-amd64.tar.gz -C /opt
[~]# mv /opt/kubernetes /opt/kubernetes-v1.16.15
[~]# ln -s /opt/kubernetes-v1.16.15 /opt/kubernetes

# 源码包,无用可删除
[/opt/kubernetes]# rm -rf kubernetes-src.tar.gz

# 镜像方式,不使用kubeadm方式可删除
[/opt/kubernetes/server/bin]# rm -rf *.tar
[/opt/kubernetes/server/bin]# rm -rf *_tag

[~]# mkdir -p /opt/kubernetes/server/bin/certs /opt/kubernetes/server/bin/conf
[~]# scp node66-103:/opt/certs/ca.pem /opt/kubernetes/server/bin/certs
[~]# scp node66-103:/opt/certs/ca-key.pem /opt/kubernetes/server/bin/certs
[~]# scp node66-103:/opt/certs/client.pem /opt/kubernetes/server/bin/certs
[~]# scp node66-103:/opt/certs/client-key.pem /opt/kubernetes/server/bin/certs
[~]# scp node66-103:/opt/certs/apiserver.pem /opt/kubernetes/server/bin/certs
[~]# scp node66-103:/opt/certs/apiserver-key.pem /opt/kubernetes/server/bin/certs

[~]# vi /opt/kubernetes/server/bin/conf/audit.yaml

apiVersion: audit.k8s.io/v1beta1
kind: Policy
omitStages:
  - "RequestReceived"
rules:
  - level: RequestResponse
    resources:
    - group: ""
      resources: ["pods"]
  - level: Metadata
    resources:
    - group: ""
      resources: ["pods/log","pods/status"]
  - level: None
    resources:
    - group: ""
      resources: ["configmaps"]
      resourceNames: ["controller-leader"]
  - level: None
    users: ["system:kube-proxy"]
    verbs: ["watch"]
    resources:
    - group: ""
      resources: ["endpoints", "services"]
  - level: None
    userGroups: ["system:authenticated"]
    nonResourceURLs:
    - "/api*"
    - "/version"
  - level: Request
    resources:
    - group: ""
      resources: ["configmaps"]
    namespaces: ["kube-system"]
  - level: Metadata
    resources:
    - group: ""
      resources: ["secrets", "configmaps"]
  - level: Request
    resources:
    - group: ""
    - group: "extensions"
  - level: Metadata
    omitStages:
      - "RequestReceived"

[~]# vi /opt/kubernetes/server/bin/kube-apiserver.sh

#!/bin/bash
./kube-apiserver \
  --apiserver-count 2 \
  --audit-log-path /export/kubernetes/kube-apiserver/audit-log \
  --audit-policy-file ./conf/audit.yaml \
  --authorization-mode RBAC \
  --client-ca-file ./certs/ca.pem \
  --requestheader-client-ca-file ./certs/ca.pem \
  --enable-admission-plugins NamespaceLifecycle,LimitRanger,ServiceAccount,DefaultStorageClass,DefaultTolerationSeconds,MutatingAdmissionWebhook,ValidatingAdmissionWebhook,ResourceQuota \
  --etcd-cafile ./certs/ca.pem \
  --etcd-certfile ./certs/client.pem \
  --etcd-keyfile ./certs/client-key.pem \
  --etcd-servers https://10.20.66.105:2379,https://10.20.66.106:2379,https://10.20.66.107:2379 \
  --service-account-key-file ./certs/ca-key.pem \
  --service-cluster-ip-range 192.168.0.0/16 \
  --service-node-port-range 3000-29999 \
  --target-ram-mb=1024 \
  --kubelet-client-certificate ./certs/client.pem \
  --kubelet-client-key ./certs/client-key.pem \
  --log-dir /export/kubernetes/kube-apiserver/logs \
  --tls-cert-file ./certs/apiserver.pem \
  --tls-private-key-file ./certs/apiserver-key.pem \
  --v 2

[~]# chmod +x /opt/kubernetes/server/bin/kube-apiserver.sh
[~]# mkdir -p /export/kubernetes/kube-apiserver/logs

[~]# vi /etc/supervisord.d/kube-apiserver.ini

[program:kube-apiserver-66-105]
command=/opt/kubernetes/server/bin/kube-apiserver.sh                           ; the program(relative uses PATH, can take args)
numprocs=1                                                                     ; number of processes copies to start (def 1)
directory=/opt/kubernetes/server/bin                                           ; directory to cwd to before exec (def no cwd)
autostart=true                                                                 ; start at supervisord start (default: true)
autorestar=true                                                                ; restart at unexpected quit (default: true)
startsecs=30                                                                   ; number of secs prog must stay running (def. 1)
startretries=3                                                                 ; max # of serial start failures (default 3)
exitcodes=0,2                                                                  ; 'expected' exit codes for process (default 0,2)
stopsignal=QUIT                                                                ; signal used to kill process (default TERM)
stopwaitsecs=10                                                                ; max num secs to wait b4 SIGKILL (default 10)
user=root                                                                      ; setuid to this UNIX account to run the program
redirect_stderr=true                                                           ; redirect proc stderr to stdout (default false)
stdout_logfile=/export/kubernetes/kube-apiserver/logs/apiserver.stdout.log     ; stdout log path, NONE for none; default AUTO
stdout_logfile_maxbytes=64MB                                                   ; max # logfile bytes b4 rotation (default 50MB)
stdout_logfile_backups=4                                                       ; # of stdout logfile backups (default 10)
stdout_capture_maxbytes=1MB                                                    ; number of bytes in 'capturemode' (default 0)
stdout_events_enable=false                                                     ; emit events on stdout writes (default false)

[~]# supervisorctl update
[~]# supervisorctl status

```

## L4反代 
```sh 
[~]# yum install -y nginx
[~]# vi /etc/nginx/nginx.conf

stream {
    upstream kube-apiserver {
        server 10.20.66.105:6443 max_fails=3 fail_timeout=30s;
        server 10.20.66.106:6443 max_fails=3 fail_timeout=30s;
        server 10.20.66.107:6443 max_fails=3 fail_timeout=30s;
    }
    server {
        listen 7443;
        proxy_connect_timeout 2s;
        proxy_timeout 900s;
        proxy_pass kube-apiserver;
    }
}

[~]# yum install -y keepalived
[~]# vi /etc/keepalived/check_port.sh

#!/bin/bash
CHK_PORT=$1
if [ -n "$CHK_PORT" ];then
  PORT_PROCESS=`ss -lnt|grep $CHK_PORT|wc -l`
  if [ $PORT_PROCESS -eq 0 ];then
    echo "Port $CHK_PORT Is Not Used,End."
    exit 1
  fi
else
  echo "Check Port Cant Be Empty!"
fi

[~]# chmod +x /etc/keepalived/check_port.sh

# master
[~]# vi /etc/keepalived/keepalived.conf

! Configuration File for keepalived
global_defs {
  router_id 10.20.66.103
}
vrrp_script chk_nginx {
  script "/etc/keepalived/check_port.sh 7443"
  interval 2
  weight -20
}
vrrp_instance VI_1 {
  state MASTER
  interface eth0
  virtual_router_id 251
  priority 100
  advert_int 1
  mcast_src_ip 10.20.66.103
  nopreempt

  authentication {
    auth_type PASS
    auth_pass 11111111
  }
  track_script {
    chk_nginx
  }
  virtual_ipaddress {
    10.20.66.1
  }
}

# slave
[~]# vi /etc/keepalived/keepalived.conf

! Configuration File for keepalived
global_defs {
  router_id 10.20.66.104
}
vrrp_script chk_nginx {
  script "/etc/keepalived/check_port.sh 7443"
  interval 2
  weight -20
}
vrrp_instance VI_1 {
  state BACKUP
  interface eth0
  virtual_router_id 251
  mcast_src_ip 10.20.66.104
  priority 90
  advert_int 1
  
  authentication {
    auth_type PASS
    auth_pass 11111111
  }
  track_script {
    chk_nginx
  }
  virtual_ipaddress {
    10.20.66.1
  }
}

[~]# systemctl start keepalived
[~]# systemctl enable keepalived

# 查看vip 10.20.66.1 是否生效
[~]# ip add
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:1e:cc:35 brd ff:ff:ff:ff:ff:ff
    inet 10.20.66.104/21 brd 10.20.71.255 scope global noprefixroute eth0
       valid_lft forever preferred_lft forever
    inet 10.20.66.1/32 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::5054:ff:fe1e:cc35/64 scope link 
       valid_lft forever preferred_lft forever

# 验证vip是否正常可以飘到slave,停掉master,查看slave
[~]# nginx -s stop
[~]# ip add

# master重启后vip飘不回来,master配置(nopreempt)非抢占式,需要停掉slave再飘回来,线上准备充分,谨慎操作
[~]# nginx
```

## kube-controller-manager
```sh
[~]# vi /opt/kubernetes/server/bin/kube-controller-manager.sh

#!/bin/sh
./kube-controller-manager \
  --cluster-cidr 172.66.0.0/16 \
  --leader-elect true \
  --log-dir /export/kubernetes/kube-controller-manager/logs \
  --master http://127.0.0.1:8080 \
  --service-account-private-key-file ./certs/ca-key.pem \
  --service-cluster-ip-range 192.168.0.0/16 \
  --root-ca-file ./certs/ca.pem \
  --v 2

[~]# chmod +x /opt/kubernetes/server/bin/kube-controller-manager.sh
[~]# mkdir -p /export/kubernetes/kube-controller-manager/logs

[~]# vi /etc/supervisord.d/kube-controller-manager.ini

[program:kube-controller-manager-66-105]
command=/opt/kubernetes/server/bin/kube-controller-manager.sh                                    ; the program(relative uses PATH, can take args)
numprocs=1                                                                                       ; number of processes copies to start (def 1)
directory=/opt/kubernetes/server/bin                                                             ; directory to cwd to before exec (def no cwd)
autostart=true                                                                                   ; start at supervisord start (default: true)
autorestar=true                                                                                  ; restart at unexpected quit (default: true)
startsecs=30                                                                                     ; number of secs prog must stay running (def. 1)
startretries=3                                                                                   ; max # of serial start failures (default 3)
exitcodes=0,2                                                                                    ; 'expected' exit codes for process (default 0,2)
stopsignal=QUIT                                                                                  ; signal used to kill process (default TERM)
stopwaitsecs=10                                                                                  ; max num secs to wait b4 SIGKILL (default 10)
user=root                                                                                        ; setuid to this UNIX account to run the program
redirect_stderr=true                                                                             ; redirect proc stderr to stdout (default false)
stdout_logfile=/export/kubernetes/kube-controller-manager/logs/controller-manager.stdout.log     ; stdout log path, NONE for none; default AUTO
stdout_logfile_maxbytes=64MB                                                                     ; max # logfile bytes b4 rotation (default 50MB)
stdout_logfile_backups=4                                                                         ; # of stdout logfile backups (default 10)
stdout_capture_maxbytes=1MB                                                                      ; number of bytes in 'capturemode' (default 0)
stdout_events_enable=false                                                                       ; emit events on stdout writes (default false)

[~]# supervisorctl update
[~]# supervisorctl status
```

## kube-scheduler
```sh
[~]# vi /opt/kubernetes/server/bin/kube-scheduler.sh

#!/bin/sh
./kube-scheduler \
  --leader-elect \
  --log-dir /export/kubernetes/kube-scheduler/logs \
  --master http://127.0.0.1:8080 \
  --v 2

[~]# chmod +x /opt/kubernetes/server/bin/kube-scheduler.sh
[~]# mkdir -p /export/kubernetes/kube-scheduler/logs

[~]# vi /etc/supervisord.d/kube-scheduler.ini

[program:kube-scheduler-66-105]
command=/opt/kubernetes/server/bin/kube-scheduler.sh                           ; the program(relative uses PATH, can take args)
numprocs=1                                                                     ; number of processes copies to start (def 1)
directory=/opt/kubernetes/server/bin                                           ; directory to cwd to before exec (def no cwd)
autostart=true                                                                 ; start at supervisord start (default: true)
autorestar=true                                                                ; restart at unexpected quit (default: true)
startsecs=30                                                                   ; number of secs prog must stay running (def. 1)
startretries=3                                                                 ; max # of serial start failures (default 3)
exitcodes=0,2                                                                  ; 'expected' exit codes for process (default 0,2)
stopsignal=QUIT                                                                ; signal used to kill process (default TERM)
stopwaitsecs=10                                                                ; max num secs to wait b4 SIGKILL (default 10)
user=root                                                                      ; setuid to this UNIX account to run the program
redirect_stderr=true                                                           ; redirect proc stderr to stdout (default false)
stdout_logfile=/export/kubernetes/kube-scheduler/logs/scheduler.stdout.log     ; stdout log path, NONE for none; default AUTO
stdout_logfile_maxbytes=64MB                                                   ; max # logfile bytes b4 rotation (default 50MB)
stdout_logfile_backups=4                                                       ; # of stdout logfile backups (default 10)
stdout_capture_maxbytes=1MB                                                    ; number of bytes in 'capturemode' (default 0)
stdout_events_enable=false                                                     ; emit events on stdout writes (default false)

[~]# supervisorctl update
[~]# supervisorctl status

# 1.16.15版本,应该是bug导致,componentstatuses显示集群状态不对，使用第二个命令查看
[~]# ln -s /opt/kubernetes/server/bin/kubectl /usr/bin/kubectl
[~]# kubectl get cs
[~]# kubectl get cs -o yaml
```
## kubelet
```sh
[~]# vi kubelet-csr.json

{
  "CN": "kubernetes-kubelet",
  "hosts": [
    "127.0.0.1",
    "10.20.66.105",
    "10.20.66.106",
    "10.20.66.107"
  ],
  "key": {
    "algo": "rsa",
    "size": 2048	
  },
  "names": [{
        "C": "CN",
        "ST": "beijing",
        "L": "beijing",
        "O": "ONLXJ",
        "OU": "ONLXJTD"
    }]
}

[~]# cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=server kubelet-csr.json | cfssl-json -bare kubelet
[~]# scp kubelet.pem root@ip:/opt/kubernetes/server/bin/certs
[~]# scp kubelet-key.pem root@ip:/opt/kubernetes/server/bin/certs

[~]# vi /opt/kubernetes/server/bin/conf/kubernetes-node.yaml

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: kubernetes-node
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:node
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: kubernetes-node

[~]# cd /opt/kubernetes/server/bin/conf

# set cluster. 证书反解 `echo ${kubelet.kubeconfig.certificate-authority-data}|base64 -d`
[~]# kubectl config set-cluster owo-cluster --certificate-authority=/opt/kubernetes/server/bin/certs/ca.pem --embed-certs=true --server=https://10.20.66.1:7443 --kubeconfig=kubelet.kubeconfig
# set-credentials
[~]# kubectl config set-credentials owo-credentials --client-certificate=/opt/kubernetes/server/bin/certs/client.pem --client-key=/opt/kubernetes/server/bin/certs/client-key.pem --embed-certs=true --kubeconfig=kubelet.kubeconfig
# set context
[~]# kubectl config set-context owo-context --cluster=owo-cluster --user=owo-credentials --kubeconfig=kubelet.kubeconfig
# use context
[~]# kubectl config use-context owo-context --kubeconfig=kubelet.kubeconfig

[~]# kubectl create -f /opt/kubernetes/server/bin/conf/kubernetes-node.yaml

[~]# scp /opt/kubernetes/server/bin/conf/kubelet.kubeconfig root@ip:/opt/kubernetes/server/bin/conf

[~]# docker pull kubernetes/pause
[~]# docker tag kubernetes/pause:latest harobr.op.com/public/pause:latest
[~]# docker push harbor.op.com/public/pause:latest

[~]# vi /opt/kubernetes/server/bin/kube-kubelet.sh

#!/bin/sh
./kubelet \
  --anonymous-auth=false \
  --cgroup-driver systemd \
  --cluster-dns 192.168.0.2 \
  --cluster-domain cluster.local \
  --runtime-cgroup=/systemd/system.slice \
  --kubelet-cgroup=/systemd/system.slice \
  --fail-swap-on="false" \
  --client-ca-file ./certs/ca.pem \
  --tls-cert-file ./certs/kubelet.pem \
  --tls-private-key-file ./certs/kubelet-key.pem \
  --hostname-override node66-105.host.com \
  --image-gc-high-threshold 20 \
  --image-gc-low-threshold 10 \
  --kubeconfig ./conf/kubelet.kubeconfig \
  --log-dir /export/kubernetes/kube-kubelet/logs \
  --pod-infra-container-image harbor.op.com/public/pause:latest \
  --root-dir /export/kubernetes/kube-kubelet

[~]# chmod +x /opt/kubernetes/server/bin/kube-kubelet.sh
[~]# mkdir -p /export/kubernetes/kube-kubelet/logs

[~]# vi /etc/supervisord.d/kube-kubelet.ini
[program:kube-kubelet-66-105]
command=/opt/kubernetes/server/bin/kube-kubelet.sh                             ; the program(relative uses PATH, can take args)
numprocs=1                                                                     ; number of processes copies to start (def 1)
directory=/opt/kubernetes/server/bin                                           ; directory to cwd to before exec (def no cwd)
autostart=true                                                                 ; start at supervisord start (default: true)
autorestar=true                                                                ; restart at unexpected quit (default: true)
startsecs=30                                                                   ; number of secs prog must stay running (def. 1)
startretries=3                                                                 ; max # of serial start failures (default 3)
exitcodes=0,2                                                                  ; 'expected' exit codes for process (default 0,2)
stopsignal=QUIT                                                                ; signal used to kill process (default TERM)
stopwaitsecs=10                                                                ; max num secs to wait b4 SIGKILL (default 10)
user=root                                                                      ; setuid to this UNIX account to run the program
redirect_stderr=true                                                           ; redirect proc stderr to stdout (default false)
stdout_logfile=/export/kubernetes/kube-kubelet/logs/kubelet.stdout.log         ; stdout log path, NONE for none; default AUTO
stdout_logfile_maxbytes=64MB                                                   ; max # logfile bytes b4 rotation (default 50MB)
stdout_logfile_backups=4                                                       ; # of stdout logfile backups (default 10)
stdout_capture_maxbytes=1MB                                                    ; number of bytes in 'capturemode' (default 0)
stdout_events_enable=false                                                     ; emit events on stdout writes (default false)

```
