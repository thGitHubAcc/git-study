##配置热部署
```sh
设置
1. File -> Settings -> Build -> Compiler 勾选 Build project automatically 
2. Ctrl + Shift + Alt + /  进入 Registry 
   勾选自动编译并调整延时参数:
   compiler.automake.allow.when.app.running -> 自动编译
   compile.document.save.trigger.delay -> 自动更新文件
   compile.document.save.trigger.delay -> 针对静态文件
3. 编辑 Run Configurations 
   On 'Update' action 选中 update classes and resources
   On frame deactivation 选中 update classes and resources

   Before launch 添加 Build  

4. maven 工程下 添加依赖
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <optional>true</optional> <!-- 这个需要为 true 热部署才有效 -->
</dependency>

```

----------------------------------------------
打开dashbord
.idea/workspace.xml添加
```xml
<component name="RunDashboard">
    <option name="configurationTypes">
      <set>
        <option value="SpringBootApplicationConfigurationType" />
      </set>
    </option>
    <option name="ruleStates">
      <list>
        <RuleState>
          <option name="name" value="ConfigurationTypeDashboardGroupingRule" />
        </RuleState>
        <RuleState>
          <option name="name" value="StatusDashboardGroupingRule" />
        </RuleState>
      </list>
    </option>
    <option name="contentProportion" value="0.2061776" />
  </component>
```





----------------------------------------------
##代码整理
ctrl + alt + I

##去掉没用的import
ctrl + alt + O

