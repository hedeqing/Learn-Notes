# 启动流程（prodef表）

## 开启流程实例

### 开启准备

```
	private ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
    private RuntimeService runtimeService = processEngine.getRuntimeService();
```

开启流程的服务初始化，使用默认引擎服务，初始化运行服务 runtimeService与运行时的操作有关

### 开启流程实例的多种方式

1. 通过流程Key来启动流程实例

   ```
   ProcessInstance processInstanve = runtimeService.startProcessInstanceByKey(流程key,变量Map)
   ```

   携带变量的使用场景，我们在设计流程图时如果设置了相关的网关等，指派人等变量时，可以通过这种方式来进行携带。而且需要和流程图设计的相符，否则流程将运行不下去,建议使用第一种,

2. 通过id来启动流程实例

```
 ProcessInstance processInstanve = runtimeService.startProcessInstanceById(流程id)
```

<u>**注：流程定义>>流程实例  这是一对多的关系，所以后续我们寻找用户任务时不用procdef来查找单个任务，而是通过流程实例id（processInstanceId）**</u>

### Map与对象的转化（转化有助于扩展更多的对象，不局限于set/get）

1. ```
   public  static HashMap<String, Object>  beanToMap(Object model) throws Exception{
       HashMap<String, Object> map = new HashMap<>();
       for (Field field : model.getClass().getDeclaredFields()) {
           field.setAccessible(true);
           map.put(field.getName(), field.get(model)); }
       return map;
   }
   ```

2. ```
   public static  <T> T mapToBean(Class<?> clazz, Map map){
       T beanInstance = (T) clazz.newInstance();
       BeanMap beanMap = BeanMap.create(beanInstance);
       beanMap.putAll(map);
       return beanInstance;
   }
   ```

   如此转化之后变量的输入就相应简单了。

   

   

