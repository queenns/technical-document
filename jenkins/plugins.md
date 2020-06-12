### 插件

#### 1. 页面操作
Manage Jenkins   =>   Manage Plugins
| Tab | Description |
| ---- | ---- |
| Updates | 可更新安装插件列表 |
| Available | 可选安装插件列表 |
| Installed | 已安装插件列表 |
| Advanced | 1.配置代理<br/>2.手动集成下载好的插件<br/>3.升级站点设置 |

#### 2. 安装插件
a. Updates选项卡选择需要安装的插件
<br/>
b. 安装时遇到超时问题重试几次
<br/>
c. Advanced选项卡更改站点，网上解决方案，超时情况好一点
<br/>
[站点设置]: (http://mirror.xmission.com/jenkins/updates/update-center.json)
<br/>
d.实际下载插件效果可能不太好，需要自己找一些源

#### 3. 基础插件
| 名称 | 用途 |
| ---- | ---- |
| Localization: Chinese (Simplified) | 汉化 |
| Credentials Plugin | Git/Harbor/etc集成时统一管理账户凭证 |
| Credentials Binding Plugin | 为变量设置适当类型的凭据 |
| Role-based Authorization Strategy | 基于角色的策略启用用户授权，其中项目矩阵授权策略操作也比较方便 |
| LDAP Plugin | LDAP方式公司账户统一登录 |
| Folders Plugin | 允许定义文件夹分类创建项目 |
| Pipeline: Job | 流水线发布模式 |
| Jenkins Git plugin | git插件 |
| Workspace Cleanup Plugin | 设置删除工作空间 |