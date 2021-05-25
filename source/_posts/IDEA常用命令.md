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