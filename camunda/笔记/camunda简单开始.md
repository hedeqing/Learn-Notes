# Camunda

## 简单开始

请详细换看官网https://docs.camunda.org/manual/latest/user-guide/process-engine/

简单学习如何开始，并且与springboot结合使用。

## camunda数据库

去导入包中可以看到对应的数据库语句

![image-20210913145932086](C:\Users\Administrator.USER-20200513YB\AppData\Roaming\Typora\typora-user-images\image-20210913145932086.png)

创建之后会有四十多张表，建议百度查看详细

## 设计流程图

看官网教程自己设计流程图（camunda model软件设计，或者网上js设计）

## 部署流程图

```
Deployment deploy = repositoryService.createDeployment()
        .addInputStream(流程key+".bpmn",in)
        .name(流程名)
        .deploy();
```

<u>`注意，如果不加“.bpmn”，则不会再procdef表上生成对应的数据，则后续将进行不下去。部署后 procdef的key是启动流程的关键所在。`</u>

## 删除流程图

如果我们想删除对应的流程图，有两种方法

1. 第一种：普通删除

```
repositoryService.deleteDeployment(部署ID)
```

2. 第二种：级联删除，会将该流程的所有历史，实例等信息全部删除，慎用

```
repositoryService.deleteDeployment(部署id, true);
```

## camunda提供的API

1. JAVA API

   笔记将从java api入手，文档可参考https://docs.camunda.org/javadoc/camunda-bpm-platform/7.15/

2. Restful API

   自己看官网，具体可参考https://docs.camunda.org/manual/7.15/reference/rest/openapi/



