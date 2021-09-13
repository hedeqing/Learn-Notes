# 任务的常规操作

## 宣称

宣称，也称认领，认领的前提是没指派人

```
taskService.claim(taskID, userId);
```

如果之前已经设定指派人，则认领失败。

```
taskService.setAssignee(task.getId(), turnId);
```

## 委托

委托别人时，状态会转为PENDING

```
taskService.delegateTask(task.getId(), userID)
```

此时，之前的指派人会成为owner，委托人会变成指派人，而后

委托人解决委托任务

```
taskService.resolveTask(task.getId());
```

此时回复原来的指派人，而状态会变成RESOLVED（不知道有没有记错）,完成之后流程并不会往下一步走，而是交给原主人，继续执行任务，主人具有决定权

## 转办

转办会转让所有权，即完成任务后不会回到转办之前的状态，会走向下一步

```
taskService.setAssignee(task.getId(), userID);
```

## 撤销

撤销具体看代码介绍

```
 public Result revoke(RevokeMessge revokeMessge) {
        String processingInstanceId = revokeMessge.getProcessInstanceId();
        String userId = revokeMessge.getUserId();
        String comment = revokeMessge.getComment();
        if (userId != null) {
            identityService.setAuthenticatedUserId(userId);
        }

        if ("COMPLETED".equals(historyService.createHistoricProcessInstanceQuery().processInstanceId(processingInstanceId).singleResult().getState())) {
            return Result.data("任务已经完成，无法撤销");
        } else {
            Task task = taskService.createTaskQuery().processInstanceId(processingInstanceId).singleResult();
            //获取当前任务，未办理任务id
            HistoricTaskInstance currTask = historyService.createHistoricTaskInstanceQuery()
                    .taskId(task.getId())
                    .singleResult();
            //获取流程实例
            ProcessInstance processInstance = runtimeService.createProcessInstanceQuery()
                    .processInstanceId(processingInstanceId)
                    .singleResult();
            //获取流程定义
            ProcessDefinitionEntity processDefinitionEntity = (ProcessDefinitionEntity) ((RepositoryServiceImpl) repositoryService)
                    .getDeployedProcessDefinition(currTask.getProcessDefinitionId());

            ActivityImpl currActivity = (processDefinitionEntity)
                    .findActivity(currTask.getTaskDefinitionKey());
            //清除当前活动出口
            List<PvmTransition> originPvmTransitionList = new ArrayList<PvmTransition>();
            List<PvmTransition> pvmTransitionList = currActivity.getOutgoingTransitions();
            for (PvmTransition pvmTransition : pvmTransitionList) {
                originPvmTransitionList.add(pvmTransition);
            }
            pvmTransitionList.clear();
            //查找上一个user task节点
            List<HistoricActivityInstance> historicActivityInstances = historyService
                    .createHistoricActivityInstanceQuery().activityType("userTask")
                    .processInstanceId(processInstance.getId())
                    .finished()
                    .orderByHistoricActivityInstanceEndTime().asc().list();
            TransitionImpl transitionImpl = null;
            if (historicActivityInstances.size() > 0) {
                ActivityImpl lastActivity = (processDefinitionEntity)
                        .findActivity(historicActivityInstances.get(0).getActivityId());
                //创建当前任务的新出口
                transitionImpl = currActivity.createOutgoingTransition(lastActivity.getId());
                transitionImpl.setDestination(lastActivity);
            }
            // 完成任务
            List<Task> tasks = taskService.createTaskQuery()
                    .processInstanceId(processInstance.getId())
                    .taskDefinitionKey(currTask.getTaskDefinitionKey()).list();
            for (Task t : tasks) {
                taskService.createComment(t.getId(), processingInstanceId, comment == null ? "" : comment);
                taskService.complete(t.getId());
//                historyService.deleteHistoricTaskInstance(t.getId());
                taskService.deleteTask(t.getId());
            }
            // 恢复方向
            currActivity.getOutgoingTransitions().remove(transitionImpl);
            for (PvmTransition pvmTransition : originPvmTransitionList) {
                pvmTransitionList.add(pvmTransition);
            }
            operationLog.setTaskId(task.getId());
            operationLog.setMethod("撤销");
            baseMapper.insertOperationMethod(operationLog);
            updateLeaveMessage(processingInstanceId);
            return Result.data(edit(processingInstanceId));
        }
    }
```

