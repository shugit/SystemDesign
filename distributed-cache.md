# Distributed Cache

## My design

[https://whimsical.com/distributed-cache-5VUUjtprvzcXZS9a1PhZuR](https://whimsical.com/distributed-cache-5VUUjtprvzcXZS9a1PhZuR)

&#x20;

## REF

[https://ravisystemdesign.substack.com/p/interview-prep-designing-a-distributed?s=w](https://ravisystemdesign.substack.com/p/interview-prep-designing-a-distributed?s=w)

&#x20;

## Business Requirement

put(key, value)

get(key)

## Non business requirement

highly available, 在硬件和网络故障情况下可用，如果要求consistency的话，这个需求可能会被替换（因为CAP理论），如果cache server fail会导致domino effect

scalable

highly performant, put和get都快

durability，当data persistent重要的情况下

security，不能对外暴露，加入一个firewall， access policy

&#x20;

## 实现

从local storage出发，讨论到distributed solution，local cahce也有用

cache可以和hsot运行在一个server或者分开，也可以使用deamon把cache包起来

&#x20;

## API Design

put(key,value)

get(key)

&#x20;

## Schema

key, value, timestamp

&#x20;

## Detailed Design

### Find cache server

use consistent hashing, each cache client uses consistent hashing to identify the cache server. Next, the cache client forwards the request to cache server maintaining a specific shard.  Each cache server have virtual servers inside it, the number of virtual servers in a machine depends on the machine's capability.  Finding a key under this algorithm requires a time complexity of O(logN), where N represents the number of cache shards.

当cache集群升级时，使用Consistent hash一致性哈希 Consistent Hashing | Algorithms You Should Know #1，或jump hash algorithm，或proportional hashing

在这个设计当中，客户端应当知道所有cache server address，实现方法可以是：设置一个固定文件存储在客户端；在S3等blob storage上设置一个文件，用户periodacally拉取这个文件；使用Zookeeper登Configuration service来注册cache service，这个方法的operational cost更高

### Improve cache server availability

Primary-replica mode. Each cache server has primary and replica servers. When primary node fails, Use a primary election algorithm to elect a new primary node among the set of available replicas. Or, use a seperate distributed configuration management service like zookeeper to monitor and select leaders. Internally, every server uses the same mechanisms to store and evict cache entries.&#x20;

### Monitoring and logging

Monitoring services such as datadog can be additionally used to log and report different metrics of cache service.

### Cache server list

Solution 1, maintaining a configuration file in each service server that cache client can use.  Each copy of configuration file can be updated through a push service by DevOps tool.

Solution 2, maintaining a configuration file on a centralized location, such as S3.

Solution 3, using a configuration service to monitor cache servers and keep cache clients updated. Configuration service ensures that all clients see an updated and consistent view of the cache servers.

### Writing policy

Async/Sync

Synchronous replication on the cache clusters will introduce a performance penalty because it takes time to replicate same data to all cache servers. After all replica finished operation, primary node will return to client. While in Asynchronous mode, primary node will return to client immediately after itself finished. Then primary node will replicate its operation to replicas. Therefore, there's a trade-off between performance and consistency in the usage of asynchronous versus synchronous replication. The answer depends on the use case.

### Eviction policy/Cache invalidation

LRU can be a good choice for social media services where recently uploaded content will likely to get the most views.

LFU is suitable for stable hotspot. can be used in e-commerce where hot items are stable.

In advertisement, LRU can be used for storing latest advertisement request, LFU can be used to store advertiser and advertising policy.

TTL value can be optimized to reduce the number of cache misses.(也是一种policy）

Scheduled Cache Eviction is a strategy that uses a separate thread to periodically remove outdated or less frequently accessed items from the cach

### Storage / Implementation

leetcode cache problem, 解法是doubly linked list和hashtable

[https://leetcode.com/problems/lru-cache/description/](https://leetcode.com/problems/lru-cache/description/)

### 缓存预热/冷启动

&#x20;

### 持久化

&#x20;

常见缓存系统区别特点

&#x20;

&#x20;

&#x20;

## Evaluation

&#x20;

&#x20;

Distinct Feature

## 一致性哈希算法

import hashlib

&#x20;

class ConsistentHashing:

&#x20;   def \_\_init\_\_(self, nodes, replication\_factor=3):

&#x20;       self.nodes = nodes

&#x20;       self.replication\_factor = replication\_factor

&#x20;       self.ring = {}

&#x20;

&#x20;       for node in nodes:

&#x20;           for i in range(replication\_factor):

&#x20;               key = self.\_get\_key(node, i)

&#x20;               hash\_val = self.\_hash(key)

&#x20;               self.ring\[hash\_val] = node

&#x20;

&#x20;   def \_get\_key(self, node, index):

&#x20;       return f"{node}-{index}"

&#x20;

&#x20;   def \_hash(self, key):

&#x20;       return int(hashlib.md5(key.encode()).hexdigest(), 16)

&#x20;

&#x20;   def get\_node(self, key):

&#x20;       hash\_val = self.\_hash(key)

&#x20;       keys = list(self.ring.keys())

&#x20;       keys.sort()

&#x20;

&#x20;       for k in keys:

&#x20;           if hash\_val <= k:

&#x20;               return self.ring\[k]

&#x20;

&#x20;       return self.ring\[keys\[0]]

&#x20;

\# 使用例子

nodes = \['cache\_server1', 'cache\_server2', 'cache\_server3']

consistent\_hash = ConsistentHashing(nodes)

&#x20;

\# 假设有100个KEY

for i in range(100):

&#x20;   key = f"key\_{i}"

&#x20;   node = consistent\_hash.get\_node(key)

&#x20;   print(f"Key '{key}' 对应的 Cache Server 是 {node}")
