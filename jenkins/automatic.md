# 自动化流程
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