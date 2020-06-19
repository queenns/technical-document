# 迁移
******
## elasticdump
1. 安装node
```sh
# curl -sL https://rpm.nodesource.com/setup_10.x | bash -
# yum install -y nodejs
```
2. 安装npm
```sh
# npm install npm@latest -g
# npm -v
6.14.5
```
3. 安装elasticdump
```sh
# npm install elasticdump -g
# elasticdump --help
version: 6.31.1
```
4. 迁移命令
```sh
#Options: [data, settings, analyzer, mapping, alias]
#
# elasticdump --input=http://127.0.0.1:9200/index --output=http://127.0.0.2:9200/index --type=settings
#
# elasticdump --input=http://127.0.0.1:9200/index --output=http://127.0.0.2:9200/index --type=mapping
```
******
## esm
1. 下载工具**https://github.com/medcl/esm/releases**
2. 迁移命令
```
# ./bin/esm -s http://127.0.0.1:9200/index -d http://127.0.0.2:9200/index -x index -w 5 -b 10 -c 1000 --refresh
```
