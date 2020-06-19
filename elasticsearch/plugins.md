# 插件
******
| 名称 | 地址 |
| --- | --- |
| analysis-ik | https://github.com/medcl/elasticsearch-analysis-ik |
| analysis-pinyin | https://github.com/medcl/elasticsearch-analysis-pinyin |
| analysis-dynamic-synonym | https://github.com/bells/elasticsearch-analysis-dynamic-synonym |
******
- analysis-ik **中文分词器**
- analysis-pinyin **拼音分析插件**
- analysis-pinyin **动态同义词插件**
******
## 安装
1. 在线安装
```sh
# ${ES_HOME}/bin/elasticsearch-plugin install ${plugin__url_package}
# ${ES_HOME}bin/elasticsearch-plugin install https://github.com/xxx/plugin-name/xxx/plugin-name.zip
```
2. 离线安装
> 1. 从github上下载对应releases的**zip**包
> 2. 未含对应版本包时下载源码自行编译打包，需要自行修改pom.xml对应版本及错误
> 3. ${HOME}/plugins目录下新建插件目录，将解压内容拷贝后重启elasticsearch即可
> 4. **http://localhost:9200/_cat/plugins**查看已安装的插件