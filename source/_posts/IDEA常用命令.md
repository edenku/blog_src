---
title: IDEA常用小技巧
date: 2021-05-11
tags: idea
---

### 如何添加RunDashBoard

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