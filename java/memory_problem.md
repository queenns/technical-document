# 内存问题排查
一、 Heap Dump文件导出
- 主动导出
  1. 确认运行时Java进程，使用ps -ef或者jps命令确认
  2. 使用jmap命令导出dump文件
    ```shell
      ## jmap -dump:format=b,file=${path}/${filename}.hprof ${pid}
      jmap -dump:format=b,file=/var/dump/service_dump.hprof 1122  
    ```
- 被动导出
  1. 设置jvm启动参数，出现OOM时自动生成dump文件
    ```shell
      -XX:-XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/var/dump/service_dump.hprof
    ```
- 容器化导出
  1. 配置持久化的卷（不合理）
  2. 通过触发点执行脚本上传dump文件至oss等
     ```shell
      -XX:OnOutOfMemoryError=./dump-shell-handler -k \$HOSTNAME -e \$ENV
     ```
  
二、Heap Dump文件分析工具
- JProfiler
  获取最大对象 -> 在图表中展示 即可分析出相关大对象及root根
- MAT
- jvisualvm