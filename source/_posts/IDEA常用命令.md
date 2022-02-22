---
title: IDEA常用小技巧
date: 2021-05-11
tags: idea
---

###  如何添加RunDashBoard

  view > Tool Windows > Run Dashboard 打开，如果没有找到，则需要在workspace.xml中添加如下代码。

  ```
  <component name="RunDashboard">
    <option name="configurationTypes">
      <set>
        <option value="SpringBootApplicationConfigurationType"/>
      </set>
    </option>
    <option name="ruleStates">
      <list>
        <RuleState>
          <option name="name" value="ConfigurationTypeDashboardGroupingRule"/>
        </RuleState>
        <RuleState>
          <option name="name" value="StatusDashboardGroupingRule"/>
        </RuleState>
      </list>
    </option>
  </component>
  ```

### Pom文件出现删除线

正常的pom文件是不会有删除线的，我们的变成这样子了。

![image-20210604094212517](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210604094212517.png)

怎么处理呢，file -> setting -> maven -> ignored Files 将被忽略的pom文件取消选中。

![image-20210604094413930](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210604094413930.png)

### IDEA 使用mvn命令时总是与配置的仓库不一致。

> 问题描述：我的项目结构是父子结构，maven项目依赖本身的子模块时，总是报模块加载异常。我mvn install后问题依然存在，后来发现，mvn install后jar包安装至D:\maven\m2\中，而与我加载的配置中设置的目录不一致。

![image-20211110112335769](https://gitee.com/ruocy/image_repo/raw/master/images/image-20211110112335769.png)

我配置的位置：

```xml
<localRepository>D:\soft\apache-maven-repo</localRepository>
```

查询发现这个可能是idea的一个bug吧。

执行mvn install时，IDEA的默认加载顺序是：

1. 优先从 ${user}/.m2 目录下读取 setting.xml
2. 当 ${user}/.m2 目录下不存在 setting.xml 时，从 ${M2_HOME}/conf 目录下读取

即使在IDEA配置了setting的位置，也不起作用。

所以删除${user}/.m2 下的setting.xml即可。