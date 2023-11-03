# 套路

## System design

This is the most conceptual part of the technical interview, and probably the most difficult. Designing data architectures is one of the most impactful tasks of data engineers. In this part, you will be asked to design a data solution from end to end, which normally comprises three aspects: data storage, data processing, and data modeling.

&#x20;

Given the rapidly growing scope of data science ecosystems, the options for design are endless. You need to be ready to discuss the pros and cons and the possible trade-offs of your choices.

&#x20;

Once you have completed the technical part, the last step of the data engineering interview will consist of a personal interview with one or more of your prospective team members. The goal? To discover who you are and how you would fit in the team.

&#x20;

But remember the data engineer interview is a two-sided conversation, meaning that you should also pose questions to them to determine whether you could see yourself as a part of the team.

一共40分钟

【3分钟】理解需求 （询问系统的商业目的 + 询问系统的功能和技术需求 + 定义成功）

【0-5分钟】资源需求（optional)（计算需要多少台机器，需要多少内存硬盘和CPU的能力）clarify requirements, QPS, storage, bandwidth estimation

【5分钟】High-level diagram, high level design

【5分钟】数据结构与存储

【10分钟】核心子服务设计component detail design

【5分钟】接口设计

【5分钟】扩展性，容错性，延迟要求, tradeoff, bottleneck, single point failure, scalability

【2-7分钟】专题 deep dive

&#x20;

&#x20;

多讨论tradeoff，多讨论pros and cons

## 答题套路

## RESHADED

requirements, estimation, storage schema, high-level design, API design, detailed design, evaluation, distinctive component/feature

### 1 理解需求，描述使用场景，约束和假设

包括两部分，理解商业需求和资源需求，也可以分为functional requirement (user have a search bar）and non functional requirements (availability, reliability)

谁会使用？会怎样使用？系统的作用是什么？

系统的输入输出分别是什么？

希望处理多少数据？每秒钟多少请求？读写比率？资源需求，计算需要多少台机器，需要多少内存硬盘和CPU的能力clarify requirements, QPS, storage, bandwidth estimation,注意这里给出的数字都需要经过细心计算和澄清，不要拍脑袋想。

e.g. what is the size of data right now, at what rate the data expected to grow over time

how will the data be consumed by other subsystems or end users such as data scientist?

is the data read-heavy or write-heavy? at what portion?

do we need strict consistency of data, or will evetual consistency work?

what's the durability target of the data?

&#x20;what privacy and regulatory requirements do we require for storing and transmitting user data?

### 2 高层次设计 High level design

画出主要组件和连接

证明想法正确性

will a conventional database work, or should we use a noSQL database?

注意每个公司有不同的风格和已经使用的组件，最好根据tech blog确定我们使用什么

需要指出design的 weakness， 设计一个follow up plan to tackle them

eg.比如我们的system cannot handle ten times more load, 但we don't expect our ststem to reach that level anytime soon, we will have a monitoring system to keep a very close eye on load growth over time so that a new design can be implemented in time.

### 3 设计核心组件 Design core component, component detail design

接口设计

&#x20;

### 4 扩展设计

确认和处理瓶颈，描述可能的解决办法和代价tradeoff

负载均衡，水平扩展，缓存，数据库分片

&#x20;

&#x20;

### 5 deepdive

&#x20;4S分析法

### Scenario

Ask/Features/QPS/DAU/Interface

#### 询问需要设计哪些功能

**Step1 enumerate 把需要设计的功能都说出来**

Registry /login

User profile display/edit

Upload image/video 重要

Search重要

Post/share a tweet

Timelie/newsfeed重要

followunfollow

**Step 2 sort 说出核心功能**

Post a tweet

Timeline

News feed

Follow/unfollow

registry

#### 询问承受多少访问量有多少QPS？

#### 可以用MAU，DAU推算出QPS，一般网站MAU/2=DAU

日活跃\*每个用户平均请求次数/一天秒数 150M\*60/86400 = 100K QPS

峰值PEAK = AVG \* 3 = 300K

快速增长的产品 fast growing MAX = PEAK \* 2

Twitter MAU 330M, DAU 170M

读频率QPS = 300k

写频率 5K（每个用户每天只写1次tweet）

QPS=100 可以用笔记本做server

QPS=1K 一台web server，需要考虑single point failure

QPS=1M 1000台server 集群,考虑maintainance

一台SQL 承受是1k QPS，如果JOIN和INDEX多的话就会更慢，所以现在工程师都尽量不写JOIN，或者使用替代JOIN的库

一台noSQl casandra承受是10K QPS

一台no SQL memcached 承受是1M QPS。不支持持久化

### Service

Split/Application/Module

把大系统拆分为小服务，进行功能拆分，把同一类问题归并在一个service中

注册/登录 放在user service

Post tweet/newsfeed/timeline放到tweet service

Upload image/video 放到media service

Follow/unfollow放到friendship service，也可以放到user service

1.replay重放需求

2.merge归并需求

问题可能有：前后端耦合coupling,级联故障cascading failure，一个模块故障导致整个系统不可用

### Storage

Schema/data/SQL/NoSQL/file system

#### 1.为每个application/service选择合适的存储结构

选择用sql/no sql/文件存储结构/s3

#### 2.Schema 细化数据表结构

这一步设计表单PK,FK，列名，列的数据类型，index怎么建

### Scale

Sharding/optimize/special case

#### 1.Optimize

解决push/pull设计缺陷

pull的缺陷：最慢的部分发生在用户读请求的时候，解决方案是把DB访问之前加入cache，cache效率是sql的100\~1000倍，cache每个用户的timeline，这样可以把N次DB请求变成N次cache请求，tradeoff

更多功能设计

解决special cases

解决push缺陷

针对inactive user

#### 2.Maintenance

Robust

scalability

## 理论

### CAP理论

Consistency,Availability,partition tolerance

在一个分布式系统中，只能同时满足如下两点

&#x20;

### 可用性模式

有两种模式，分别有优缺点

#### Fail-over 故障切换

缺陷是需要添加额外的硬件并增加复杂性，如果故障发生在新写入的数据在被复制到备用系统之前，可能会丢失数据

**主从切换 active - passive**

**双工作切换/主主切换 active-active**

两个工作都管控流量，之间分散负载

&#x20;

#### replication复制

主从复制

&#x20;

主主复制
