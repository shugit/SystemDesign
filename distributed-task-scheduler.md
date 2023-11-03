# Distributed Task Scheduler

把任务信息保存到数据库中

任务推送到distributed quqeue执行

资源节点发送心跳给集群资源管理器，管理器维护任务列表和接受该任务的节点ID

&#x20;

## Business requirement

任务来自不同的sources, tenant, sub-systems

资源分塞在一个或不同的data center

提交任务

分配资源

删除任务

监控任务执行：如果失败，自动重试

effeicient resource utilization: time, cost, fairness、

释放资源

显示任务状态

&#x20;

## non-biz requirement:

availablbility=Reliable, 由Rate limiter 和replaceable node实现

durability，存储在分布式数据库中的任务信息

Scalable，分布式数据库，分布式队列，可调整的资源

fault-tolerant，如果任务包含无限循环，指定时间后终止并通知任务，如果执行失败则重试

bounded waiting time：如果等待时间超过阈值，用户应该被通知到

&#x20;

## 需使用的模块

rate limiter, sequencer, database, distributed queue, monitoring， top k priority

&#x20;

## 优先级设计

优先级基于任务属性，包括delay tolerance, short execution cap，top k被推入distributed queue，k可以根据许多因素比如当前可用资源来计算

### 分类

Cant delayed task

can delayed task

executed periodically task(每月，每周，每天，每小时）

\*如果低优先级任务马上超时，将会被移入到高优先级队列 << 什么算法？

&#x20;

任务的识别：幂等key, idempotency key

&#x20;

## Execution Cap

确定execuetion的算法：如何设置，由用户定义？

怎么防止恶意资源竞争？

max allowed attempt
