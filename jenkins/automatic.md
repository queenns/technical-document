# 自动化构建项目
基于Spring-Boot项目类型
******
## Jenkins创建Job
******
1. Prepare an environment for the run（Environment Injector Plugin）
- Keep Jenkins Environment Variables
- Keep Jenkins Build Variables
2. 参数化构建过程

| 名称 | 描述 | 类型 |
|:--- |:--- |:--- |
| git_url | 例：https://github.com/queenns/technical-document.git| 字符参数 |
| git_branch_or_tag | 分支名称/tag名称 | 字符参数 |
| git_sub_module | 子目录，项目结构含子项目 | 字符参数 |
| cluster | 集群环境：dev/test/pre/prod | 字符参数 |
| mvn_cmd | 编译命令 | 字符参数 |
3. 源码管理（Jenkins Git plugin）

- Git - Repositories - Repository URL，${git_url}
- Git - Repositories - Credentials，Git凭证
- Git - Branches to build - ${git_branch_or_tag}
4. 构建环境（Credentials Binding Plugin）

- Delete workspace before build starts
- Use secret text(s) or file(s)
5. 绑定

- | 类型                             | 用户名变量   | 密码变量     | 凭据方式 | 凭据                                 |
| :------------------------------- | :----------- | :----------- | :------- | :----------------------------------- |
| Username and password(separated) | HARBOR_USER  | HARBOR_PASS  | 指定凭据 | Harbor凭证                           |
| Username and password(separated) | JENKINS_USER | JENKINS_PASS | 指定凭据 | Jenkins凭证，凭证中的密码是用户token |
- Set jenkins user build variables
6. 构建

- 执行 shell
```sh
source /var/lib/jenkins/.bashrc
sh /var/lib/jenkins/tools/jenkins/init.sh
```
******
## 服务器环境
******
1. Docker环境安装
2. pip安装
```sh
# yum -y install python-pip
```
3. python-jenkins安装
```sh
# pip install python-jenkins
```
4. python工具类**/var/lib/jenkins/tools/utils/StringUtil.py**
```python
def set_sys_default_encode(default_encoding):
    if sys.getdefaultencoding() != default_encoding:
        reload(sys)
        sys.setdefaultencoding(default_encoding)
def set_sys_utf8():
    set_sys_default_encode('utf-8')
```
5. maven安装
```sh
# tar -xvzf apache-maven-3.5.4-bin.tar.gz
# mv apache-maven-3.5.4 maven
```
6. 环境变量配置
```sh
# vim /var/lib/jenkins/.bashrc
export M2_HOME=/root/maven
export PATH=$M2_HOME/bin:$PATH
export PYTHONPATH=$PYTHONPATH:/var/lib/jenkins/tools/utils
```
7. rancher可执行二进制文件准备**/var/lib/jenkins/tools/rancher/rancher**
```sh
# ls -al
total 9216
drwxr-xr-x 3 jenkins jenkins    4096 Jun  9 18:24 .
drwxr-xr-x 5 jenkins jenkins    4096 Jun 11 13:46 ..
-rwxr--r-- 1 jenkins jenkins     570 Jun  9 18:24 flatten.py
-rwxr-xr-x 1 jenkins jenkins 9417770 Jun  5 14:26 rancher
drwxr-xr-x 2 jenkins jenkins    4096 Jun  8 11:26 tokens
```
8. rancher**/var/lib/jenkins/tools/rancher/tokens/test.conf**准备
```sh
# cat test.conf 
# profile
profile_name=dev
# rancher
context="local:p-xxxxxx"
url="https://rancher.xcar.com.cn/v3"
# rancher login valid user
jenkins_token=token-c2mlm:vgjc7shpxxxxxxzwjpc29q42727lffw6vr9hd2pfl7fx7bl5xxxxxx
# rancher user token
liu_xiaojian=token-czw8x:dg95vl9pxxxxxx9pvzmz9h95gp8crnwttr9tlswfjv75stmkxxxxxx
```
9. 数据转化脚本**/var/lib/jenkins/tools/rancher/flatten.py**
```python
#! /usr/bin/env python
import sys
from yaml import load
import json
def flatten_json(y):
    out = {}

    def flatten(x, name=''):
        if type(x) is dict:
            for a in x:
                flatten(x[a], name + a + '.')
        elif type(x) is list:
            i = 0
            for a in x:
                flatten(a, name + '['+str(i) + '].')
                i += 1
        else:
            out[name[:-1]] = x

    flatten(y)
    return out
with open(sys.argv[1]) as f:
     yml_dic=load(f)
     print json.dumps(flatten_json(yml_dic)).replace('.[','[')
```
10. 初始化脚本**init.sh**
```sh
#!/usr/bin/env bash
set -e

# 1.执行构建发布流程
dir=$(pwd)
build="sh /var/lib/jenkins/tools/jenkins/build.sh ${dir}/${gitlab_sub_module}"
eval ${build}

# 2.自动化创建Job
cmd="/bin/python ${HOME}/tools/jenkins/jenkins-automatic.py \
--git_url=${git_url} \
--sub_module=${gitlab_sub_module} \
--jenkins_url=http://localhost:8080 \
--jenkins_user=${JENKINS_USER} \
--jenkins_auth=${JENKINS_PASS}"
eval $cmd
```
11. 构建发布脚本**build.sh**
```sh
#!/usr/bin/sh
set -e
source /var/lib/jenkins/.bashrc

# 1.如果是子模块项目，切换到子模块运行
sub_path=${1:-'.'}
cd ${sub_path}

# 2.注入Rancher执行路径，简化执行命令
export RANCHER_HOME="/var/lib/jenkins/tools/rancher"
export PATH=$PATH:${RANCHER_HOME}

# 3.解析helm
export helm_file=$(ls .helm-*.yaml| head -n 1)
export helm_name=$(echo ${helm_file} |awk -F '.' '{print $2}'| sed 's/helm-//g')

export NAMESPACE=$(cat ${helm_file}| grep '^namespace:' |gawk  '{print $2}')
export APP=$(cat ${helm_file}| grep '^name:' |gawk  '{print $2}')
export DOCKER_IMAGE=$(cat ${helm_file} | grep '^image:' |gawk '{print $2}')
export HOST_NAME=$(cat ${helm_file} | grep '^  host: ' |gawk '{print $2}')
export JENKINS_PATH=$(cat ${helm_file} | grep '^  project_path: ' |gawk '{print $2}')

# 4.Rancher的Jenkins管理员用户登录，判断新建还是更新
source ${RANCHER_HOME}/tokens/${CLUSTER}.conf
login=$(echo "rancher login ${url} -t ${jenkins_token} --context ${context}"|tr "\r" " ")
echo $login
eval $login

namespace_exist=1
for ctx in $(rancher projects ls -q| awk '{print $1}')
do
  rancher context switch ${ctx}
  if [ $(rancher  namespaces -q | grep -w ${NAMESPACE})0 = ${NAMESPACE}0 ]; then
       export user_context=${ctx}
       namespace_exist=0
       break
  fi
done
if [[ namespace_exist -eq 1 ]];then
    echo "命名空间不存在，请先创建"
    exit 1
fi

# 5.执行maven编译
eval $MVN_CMD

# 6.镜像打包及处理
tag=$(echo ${GIT_COMMIT} | cut -c 1-8)
if [[ ${BUILD_DOCKER_WITH_GIT_TAG} = "true" ]]; then tag=${branche_or_tag} ; fi
image=${DOCKER_IMAGE}:${tag}
build_cmd="docker build --build-arg APP_NAME=${APP} -t ${image}  ."
if [ ${helm_name} = "tomcat" ]
then
  war_name=$(cat ${helm_file}| grep '^war_name:' |gawk  '{print $2}')
  build_cmd="docker build --build-arg WAR_NAME=${war_name} -t ${image}  ."
fi
echo ${build_cmd}
eval ${build_cmd}
docker login -u${HARBOR_USER} -p${HARBOR_PASS} data-harbor.xcar.com.cn
docker push ${image}
docker rmi ${image}

# 7.rancher部署
export user_token_key=$(echo  ${BUILD_USER_ID} | tr "." "_")
export user_token=$(cat ${RANCHER_HOME}/tokens/${CLUSTER}.conf | grep "^${user_token_key}=" |gawk -F "="  '{print $2}')

tmp_answers_file=/tmp/values.json
flatten.py ${helm_file}  > ${tmp_answers_file}

values=""
if [ ${helm_name} = "spring-boot" ]
then
    values="--set env.profile_name=${profile_name}"
fi

answers="--answers ${tmp_answers_file} --set author=${BUILD_USER_ID}  --set image=${image} ${values} ${APP} "
install="rancher apps install cattle-global-data:xcar-${helm_name} --version 0.1.0 -n ${NAMESPACE} ${answers}"
upgrade="rancher apps upgrade ${answers} 0.1.0"

cmd=${install}
if [[ $(rancher apps -o json| jq -r .App.name | grep -w ${APP})0 = ${APP}0 ]]; then
    cmd=${upgrade}
fi
user_login=$(echo "rancher login ${url} -t ${user_token} --context ${user_context}"|tr "\r" " ")

eval ${user_login}
eval ${cmd}
```
12. 自动化创建Job脚本**jenkins-automatic.py**，配置初始项目，读取config后替换部署项目的各种值
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
import argparse
import os
import sys
import jenkins
import StringUtil
from yaml import load

StringUtil.set_sys_utf8()

args = None


def get_parser():
    parser = argparse.ArgumentParser()
    parser.add_argument('--gitlab_url', )
    parser.add_argument('--sub_module', default='')
    parser.add_argument('--jenkins_url')
    parser.add_argument('--jenkins_user')
    parser.add_argument('--jenkins_auth')
    return parser


def validate():
    if args.sub_module:
        args.sub_module = args.sub_module.lstrip("/")


def make_folder(server, dirs):
    tmp = ''
    floder_config = server.get_job_config('xcardata/demo')
    for dir_ in dirs:
        tmp += dir_ + '/'
        if not server.job_exists(tmp[:-1]):
            print '创建目录:' + tmp[:-1]
            server.create_job(tmp[:-1], floder_config)


def create_config(server):
    old_project_name = 'k8s-demon-helloworld' + ('-submodules' if args.sub_module else '') + "-1.0"
    config = server.get_job_config('xcardata/demo/' + old_project_name)

    DEFAULT_GIT_URL = 'http://gitlab.xcar.com.cn/data-devops/spring-boot-k8s-demo.git'
    config = config.replace(DEFAULT_GIT_URL, args.gitlab_url)

    env_dist = os.environ
    logger.info('sys env env_dist.get(MVN_CMD):%s' % env_dist.get("MVN_CMD"))
    if args.sub_module:
        DEFAULT_INVOKE_SHELL = '<command>sh /var/lib/jenkins/tools/jenkins/build.sh submodules</command>'
        REPLACE_INVOKE_SHELL = '<command>%s \n sh /var/lib/jenkins/tools/jenkins/build.sh %s</command>' % ('export MVN_CMD=' + '\"' + env_dist.get("MVN_CMD").replace("&", "&amp;") + '\"', args.sub_module)
        config = config.replace(DEFAULT_INVOKE_SHELL, REPLACE_INVOKE_SHELL)
    else:
        DEFAULT_INVOKE_SHELL = '<command>sh /var/lib/jenkins/tools/jenkins/build.sh</command>'
        REPLACE_INVOKE_SHELL = '<command>%s \n sh /var/lib/jenkins/tools/jenkins/build.sh</command>' % ('export MVN_CMD=' + '\"' + env_dist.get("MVN_CMD").replace("&", "&amp;") + '\"')
        config = config.replace(DEFAULT_INVOKE_SHELL, REPLACE_INVOKE_SHELL)

    logger.info('DEFAULT CONFIG: %s' % config)

    return config


def main():
    server = jenkins.Jenkins(args.jenkins_url, username=args.jenkins_user, password=args.jenkins_auth)

    helm_conf = None
    for dir in os.listdir('./' + args.sub_module):
        print('dir %s' % dir)
        if dir.startswith('.helm-'):
            with open('./' + args.sub_module + '/' + dir) as file:
                helm_conf = load(file)

    jenkins_conf = helm_conf.get('jenkins')
    if jenkins_conf:
        project_path = jenkins_conf['project_path']
    else:
        print('.helm-spring-boot.yaml没有Jenkins相关配置')
        sys.exit(1)

    exists = server.job_exists(project_path)
    print('%s exists: %s' % (project_path, exists))

    dirs = project_path.replace('\\', '/').strip().rstrip('/').split('/')[:-1]
    print('dirs %s' % dirs)
    make_folder(server, dirs)

    config = create_config(server)
    
    if exists:
        server.reconfig_job(project_path, config)
        print('%s 更新成功', project_path)
    else:
        server.create_job(project_path, config)
        print('%s 创建成功', project_path)


if __name__ == '__main__':
    args = get_parser().parse_args()
    validate()
    main()
```