# Jenkins

官方地址：[https://www.jenkins.io/](https://www.jenkins.io/)
******
持续集成、自动化构建部署软件代码工具
******
**CI/CD**
- ***CI***   持续集成(Continuous Integration) 
- ***CD***  持续交付(Continuous Delivery)
******
![Jenkins](images/jenkins.jpg)
******
**工作结构流程图**
| 角色 | 功能 |
| :--- | :--- |
| Developer | 开发者 |
| Tester | 测试员 |
| Code Store | 代码仓库：Git Hub、Git Lab、 SVN |
| Server | 服务器：物理机、Docker、Kubernetes |
******
1. 开发者根据需求将最新代码提交到代码仓库

2. 开发者/测试员页面操作Jenkins，提交发布构建任务(submit job)

3. **Jenkins内部**检出源代码，一般通过Git插件等方式

4. **Jenkins内部**编译代码，一般编写shell脚本根据不同编程语言进行配套的编译工作

5. **Jenkins内部**处理Docker镜像，如果使用镜像方式部署

6. **Jenkins内部**部署，将程序包上传至服务器并脚本启动**/**脚本将镜像在容器环境发布
******
**概念**：第3-6步骤在Jenkins中被称作**Job**，**Job**是一个完整的发布流程，**Job**由（组件+脚本）组合实现
******
## 目录
* [安装](install.md)
* [插件](plugins.md) 