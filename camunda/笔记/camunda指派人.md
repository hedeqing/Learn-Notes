# 任务指派（task表）

开启流程之后会有用户任务，需要指派任务人来完成

## 开始准备

```
private TaskService taskService = processEngine.getTaskService();
```

任务管理服务操作通过这个服务来启动，

## 查找任务

查找多个任务与单个任务有所不同，观察表来进行选择，

1. 单个选择，可以找唯一属性来筛选唯一task

```
Task task = taskService.createTaskQuery()
        .processInstanceId(processInstance.getID())
        .singleResult();
```

2. 多个任务选择，如通过流程定义Key来进行筛选

```
List<Task> taskList = taskService.createTaskQuery()
        .processDefinitionKey(key)
        .list();
```

### 指派任务的多种方法

1. 这个可以在设计流程图时设计assign变量，然后开启流程时使用变量输入来完成指派

```
HashMap<String, String> variables = new HashMap<>();
variables.put("设置的assign变量名", "指派人id");
```

官网指派人事字符串型的唯一值，但是我们日常使用的是Long类型的id作为我们的用户id，所以建议还是使用唯一的id来进行指派即可，查找时也更加方便

2. 实现不指定任务指派变量，开启流程之后通过

   ```
   taskService.setAssignee(task.getID(),指派人 )
   ```

   来进行指派，不建议使用

```
	task.setAssignee(指派人id)
```

其中缘由我也不太懂，<u>前者能实际指派上，后者指派不上</u>

## 网关前的指派任务人选择

实际开发中，我们需要在网关之前的用户任务就指定下一步的指派人，但是引擎需要通过网关的表达式条件才知道我们的下一步王哪里走，但是我们又需要前端知道好指派下一步指派人，所以这时候我们就需要读取网关的判断条件来提前进行判断

1. 遍所有节点找到网关节点

```
ProcessDefinitionEntity processDefinitionEntity = (ProcessDefinitionEntity) ((RepositoryServiceImpl) repositoryService)
                .getDeployedProcessDefinition(definitionId);
List<ActivityImpl> activitiList = processDefinitionEntity.getActivities();
                

```

2. 活动节点中筛选网关节点

"exclusiveGateway".equals(activity.getProperty("type"))

3. 获取节点的网关节点的所有通路（出口）

   ```
   List<PvmTransition> outTransitions = activity.getOutgoingTransitions();
   ```

4. 遍历所有通路获取所有通路的判断条件

```
Object s = ots.getProperty("conditionText");
```

5.  获取表达式之后带入下面判断，key:设置的条件变量值，el：表达式（也就是s），value：我们输入的值

```
public boolean isCondition(String key, String el, String value) {
    ExpressionFactory factory = new ExpressionFactoryImpl();
    SimpleContext context = new SimpleContext();
    context.setVariable(key, factory.createValueExpression(value, String.class));
    ValueExpression e = factory.createValueExpression(context, el, boolean.class);
    //特别注意在这个位置需要设置gateWay的Id值–这个Id值与gatway流出的flow的el表达式的key要对应（不然程序在执行中会出错）
    return (Boolean) e.getValue(context);
}
```

6. 返回为true的通路就是我们下一步的走向，指派人也就能提前解决了。
