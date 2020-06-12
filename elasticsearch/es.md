# Elasticsearch

1. 安装包：[elasticsearch-7.2.0-linux-x86\_64.tar.gz](https://pan.baidu.com/s/1OsCx1dRUtvtBYUSgCwUKOw) 提取码: gpej
2. 官方下载：[https://www.elastic.co/cn/downloads/elasticsearch](https://www.elastic.co/cn/downloads/elasticsearch)

## 安装步骤

1. 操作系统
   ```bash
   root@root:~$ cat /etc/redhat-release 
   CentOS Linux release 7.5.1804 (Core)
   ```
2. Java环境**Java8+**
   ```bash
   root@root:~$ java -version
   openjdk version "1.8.0_232"
   OpenJDK Runtime Environment (build 1.8.0_232-b09)
   OpenJDK 64-Bit Server VM (build 25.232-b09, mixed mode)
   ```
3. 安装包拷贝至 **/opt** 目录
   ```bash
   root@root:/opt$ ls -al
   drwxr-xr-x 13 root root 4096 Mar   7 10:24 .
   drwxr-xr-x 27 root root 4096 Sep  14  2020 ..
   drwxr-xr-x  4 root root 4096 Jul  28  2020 elasticsearch-7.2.0-linux-x86_64.tar.gz
   ```

4. 解压安装包

   ```bash
   root@root:/opt$ tar -xvf elasticsearch-7.2.0-linux-x86_64.tar.gz
   ```

5. 添加用户

   ```bash
   root@root:/opt$ useradd -M -s /sbin/nologin elastic
   ```

6. 修改es权限

   ```bash
   root@root:/opt$ chown -R elastic:elastic elasticsearch-7.2.0/
   ```

7. **/opt/elasticsearch-7.2.0/config/jvm.options** 根据实际情况设置堆内存大小

   ```text
   # Xms represents the initial size of total heap space
   # Xmx represents the maximum size of total heap space

   -Xms512M
   -Xmx512M
   ```

   8.**/opt/elasticsearch-7.2.0/config/elasticsearch.yml**

   ```yaml
   # 集群名称
   cluster.name: local-cluster
   # 节点名称
   node.name: local-elasticsearch-node
   # 存储数据目录路径
   path.data: /export/elasticsearch/data
   # 存储日志目录路径
   path.logs: /export/elasticsearch/logs
   # 是否禁用swap交换分区,不禁用的话会导致IOPS变高,性能下降,直接使用物理内存保证性能
   bootstrap.memory_lock: false
   # 将绑定地址设置为特定IP(IPv4或IPv6),ipconfig查看本地ip设置即可
   network.host: 10.200.16.4
   # http访问端口
   http.port: 9201
   # 节点交互端口
   transport.port: 9301
   # 传递要执行发现的主机的初始列表,集群模式配置所有节点
   discovery.seed_hosts: ["local-elasticsearch-node:9301"]
   # 符合主节点条件的节点列表
   cluster.initial_master_nodes: ["local-elasticsearch-node"]
   # N节点启动后集群开始初始恢复
   gateway.recover_after_nodes: 1
   ```

8. 配置本机Ip的host **/etc/hosts**

   ```bash
   echo '10.200.16.4 local-elasticsearch-node' >> /etc/hosts
   ```

9. 设置**bootstrap.memory\_lock=true**时，需要修改服务器配置

   | 命令 | 描述 |
   | :--- | :--- |
   | \* | 表示所有用户 |
   | soft | 代表警告的设定，可以超过这个设定值，但是超过后会有警告 |
   | hard | 代表严格的设定，不允许超过这个设定的值 |
   | nofile | 是每个进程可以打开的文件数的限制 |
   | nproc | 是操作系统级别对每个用户创建的进程数的限制 |
   | memlock | 是操作系统级别对每个用户内存的限制 |

   ```bash
   sudo vi /etc/security/limits.conf

   # 添加以下内容

   * soft nofile 65536
   * hard nofile 65536
   * soft nproc 2048
   * hard nproc 2048
   elastic soft memlock unlimited
   elastic hard memlock unlimited
   ```

   设置进程能拥有的最多内存区域

   ```bash
   echo 'vm.max_map_count = 262144' >> /etc/sysctl.conf
   sysctl -p
   ```

10. 启动

    ```bash
    sudo su - elastic
    nohup /opt/elasticsearch-7.2.0/bin/elasticsearch > /dev/null 2>&1 &
    ```

