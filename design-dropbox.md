# Design Dropbox

## Ref

[Design Google Drive or Dropbox (Cloud File Sharing Service) | System Design Interview Prep](https://www.youtube.com/watch?v=jLM1nGgsT-I)

&#x20;

## My Design

[https://whimsical.com/design-dropbox-M4pL5yvmqPDZMyy8BoiDB4](https://whimsical.com/design-dropbox-M4pL5yvmqPDZMyy8BoiDB4)

## Business requirement

upload file with conflict management

download file

sync throught multiple devices

notification when new file is uploaded

## Non business requirement

availability

scalability

consistency

highly performant

reliability

&#x20;

## API Design

box.com/upload?filename=\&filesize=\&timestamp=

box.com/download?filename=

Sync using long polling to transfer data

{ filename:"", size:"", update\_time:"",action\_type:"new/remove/edit",  content:""}

notification when new file is uploaded

{ filename:"", size:"", update\_time:"" ,action\_type:"new/remove/edit"}

box.com/share?filename=

&#x20;

## Schema Design

File metadata

user\_id, file\_id, filename, filesize, create\_time, updated\_time, is\_removed

Block metadata

file\_id, block\_seq, location, is\_removed

考虑到相同block的的设计，需要更新

&#x20;

&#x20;

## High level design

## Estimation

Assumption

Sign-up user:50million

DAU:10million

Space for each user: 10GB

Read-to-write ratio: 1:1

Average file size: 10KB

daily uploaded file: 2 files

size for each file in DB: 1K

file number for each user: 1000

&#x20;

Calculation

Space needed: 50 million \* 10G = 500PB, 计算方式是10G \* 1000 = 10TB， 10TB \* 1000=10PB， 10PB\*50 = 500PB

QPS for upload/download: 2 \* 10million / seconds in day = 20 mil / (2600\*24) = 20 mi / 86.4K  = 0.23K = 240 qps

Peak QPS = QPS\*2 = 480

Bandwidth: 10KB\*2\*2\*10million / 86400 = 40K \* 10 million / 86.4K = 400/86.4 Million = 4.6MB = 36.8Mbps

Peak Bandwidth = 73.6Mbps

DB Usage: 50 million \* 1000 \* 1KB =  50 million \* 1MB = 50TB

message queue usage: 2 \* 1K \* 10million = 20 million K=  20G

It's a IO intense application

&#x20;

## Detailed Design

我们先设计正常流程和主备方案，然后讨论每个步骤fail的处理方式

LoadBalancer: Primary-Secondary mode。

If a load balancer fails, the secondary would become active and

pick up the traffic. Load balancers usually monitor each other using a heartbeat, a periodic

signal sent between load balancers. A load balancer is considered as failed if it has not sent

a heartbeat for some time.

API Server: Primary-secondary mode

It is a stateless service. If an API server fails, the traffic is redirected

to other API servers by a load balancer.

用户请求为upload: api server首先会根据文件大小分配相应的block server(分配算法待定)，返回给用户的客户端一个列表{block\_seq, location} ， 用户根据返回的列表和它对应的BLock server直接沟通上传文件Block， 根据Dropbox的设计，每个block大小是4M， 所以一个512M的文件会被分割为128个block， api server会写到metadata cache和db（注意此处cache和db的update顺序，一旦update fail了怎么设计）的block table和file table中{filename, status:"uploading"}和{block\_seq, location, status:"uploading"}用来标记block正在被上传，每个block上传完成后会标记相应的block status:"finished"，当所有block都上传完成后api server在file table里标记status='complete'，以确保整个区块一起可见。此时api server会把notification写入到message queue {filename, filesize, action='new\_file'}。

此处设计需要讨论的tradeoff地方是，block server尽量分配在相同的机器还是尽可能在不同机器？是否尽可能写入到连续的磁盘空间？如果在连续磁盘空间，那么查找到的速度会变快，但是数据传输速度受到单机限制，如果多个用户同时请求该文件，则速度会变慢，多个server的状况可以使得该文件获得更多的带宽，但是io开销较大，因为磁盘读取不是顺序的了。

同时block size也可以优化，如果我们的box是以照片为主，那么4mb会合适，但如果是文档为主，100Kb可能会更合适。

是否blockserver replica都写入在返回，还是写入一个master就反悔，这是一个上传速度vs reliabity tradeoff，我认为应该取后者。比起慢一点，丢失文件是更不能接受的。

metadata cache：可以使用redis或rabbit Q， redis可以使用redis cluster用于sharding，此处可以使用consistent hash根据user id做hash到相应的redis cluster， redis可以同时开启master-slave模式，slave用于读取，master接受到read之后replicicate到slave，这里我们使用redis默认的async复制模式。（考虑redis冷启动）redis sentinel模式可以开启至少3个运行在redis server上，这样一旦有master有问题之后，其他2个slave可以投票选出新的master，通知通知连接的api server这个master/slave状态变化。

此处也有sync or async问题，以及master问题，节点数量问题（保持奇数以更好选出领导防脑裂）

以及cache invalidation策略，LRU，LfU，定时，还有cache db先后问题。都分别讨论trade off。

metadata DB： 可以使用MYSQL， 把redis中的信息持久化，mysql也需要使用相应的sharding和replica方案，我会推荐将mysql的数量设置为redis的倍数，并且考虑consistent hashing时 redis

MYsql 是n对1关系，这就意味着每台api server只需要维持长连接到尽可能少且固定的redis和mysql数据库，比如1 api server对8台redis，对4台mysql，这样api server和数据库的连接不是mesh形式的，减少连接可以有助于提升redis和mysql的性能，同时保证查找的时候磁盘不会跳跃太多分区。 api server会保持1个write连接到mysql master，保持1个read 连接到mysql slave。master接受到write之后写入到slave，只有当write写入到slave都完成之后我们才会返回success。

Block server: block server我们会使用更多的磁盘和更少的CPU core的机型，我们会建立对于这些sever的监控查看CPU allocation , network allocation and disk allocation，以确保降低资源开销。 block server接受用户的文件写入，也返回给用户文件的block，根据hdfs的默认配置，我们将replica设为3，也就是说每个block会被写入到3台机器上，在这里我们使用strong consistency的sync replica模式，只有复制全部完成后才会返回成功。如果为了增强region availabibility，我们可以使用2台本region replica，一台跨region replica，如果是这个方案的话，那么本region replica成功之后就返回，因为考虑到跨region的网络带宽问题，过长时间的replica可能给用户带来不好的体验。

文件去重：由于用于可能上传相同的文件（比如电子书，电影），我们可以在上传时为文件检查一次md5编码，查看是否是相同的文件。此处新增一个table {user\_id, file\_id, origin\_file\_id}做一个映射，即使文件被原上传人删除，文件仍保留不变，映射关系做一个更改，将第二个上传人设为primary file owner，直到该文件在没有对应的owner，我们才标记为"to be removed"，等待系统清除。Md5碰撞:我们使用md5和sha256，只有这两个编码都相同才认为是相同文件。

&#x20;

block去重：由于用户每次编辑可能只编辑少量字符，我们可以检查该文件被编辑的block，只重新上传被编辑的block。block设计是immutable的，新上传的block成功之后我们才会给之前的block做status:"to\_be\_removed"标记。不同文件可能block会相同，需要重新设计schema。我们可以为相同blcok做md5+sha256，这里就有一个tradeoff。我们是要更细粒度的match，以节约更多的存储空间，但消耗更多的db空间，以及更大的颗粒度，在文件上做md5，以节约db空间，并且对于相同文件的处理速度会更快。我们可以根据不同文件类型做相应调整。

&#x20;

文件删除：当用户删除文件的时候，我们为该文件在cache和db里面都标记状态为"to be removed"，并保证不可见，由于每个文件被指定了一个file\_id，即使相同文件名再次上传依然可以获得新的file\_id，用户如果再次请求该文件，由于该文件已经被标记为删除，用户依然不可见。在流量低估时期，我们会对block server,cache, mysql做一次clean，把被标记的文件从磁盘中删除。

notification设计：当文件被上传，编辑或删除的时候，api server发送给message queue一条消息，表示文件有变化，这里的message queue我们可以设计为distributed message queue，依然根据consistent hashing根据user id把消息发送到相应的queue sharding，notification sever和用户会建立long polling（根据dropbox设计）， 当用户在线时则推送到用户client，并把message相应的message删除，如果用户此时不在线，则复制消息到mysql表并删除message， schema {user\_id, file\_name, action, timestamp}，待用户上线时从mysql中查询。我们要保证消息的发送速度大于消息的产生速度，否则会造成消息积压。distributed message queue也会使用sharding来保证相同user的消息都在一个partition内，同时我们根据kafka默认配置相同的replica=3的配置，server1保存p1,p2(r), server2保存p2,p1(r), server3保存p1(r),p2(r)。写入消息时我们的consistency不需要很高，只要master获得消息成功了便可以返回，就是async模式，distribued message queue使用zookeeper来协调master/replica配置。

Additionally，我们可以把经常被拉取的块放在CDN上作为缓存

&#x20;

## Evaluation

availability，每个component都有备份节点

scalability， 通过sharding实现

consistency， db和cache更新要保持一致

highly performant，通过CDN，cache实现

reliability， 每个文件都被备份3次可以实现



需要弄清楚的：

MYSQL ,REDIS sharding和master-slave, primary-seconday配置方法

[https://www.cnblogs.com/you-men/p/13113511.html](https://www.cnblogs.com/you-men/p/13113511.html)

API server基本的方法

考虑redis冷启动

api server首先会根据文件大小分配相应的block server的分配算法待定

注意此处cache和db的update顺序，一旦update fail了怎么设计，数据库与缓存同步是一个常见难题 [https://pandaychen.github.io/2022/01/01/A-REDIS-MYSQL-KEEP-CONSISTENT-METHOD/](https://pandaychen.github.io/2022/01/01/A-REDIS-MYSQL-KEEP-CONSISTENT-METHOD/) [https://blog.csdn.net/sufu1065/article/details/108459758](https://blog.csdn.net/sufu1065/article/details/108459758)

&#x20;
