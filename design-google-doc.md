# Design Google Doc

这一章比较多可以讨论关于分布式系统中的合并算法

&#x20;

## ref

[https://blog.bytebytego.com/p/how-to-design-google-docs-episode](https://blog.bytebytego.com/p/how-to-design-google-docs-episode)

[https://www.linkedin.com/pulse/google-docs-high-level-system-design-murat-atak/](https://www.linkedin.com/pulse/google-docs-high-level-system-design-murat-atak/)

&#x20;

## My design

[https://whimsical.com/google-doc-RMkeEZaQKZP95CUmkdTa17](https://whimsical.com/google-doc-RMkeEZaQKZP95CUmkdTa17)

&#x20;

## Business requirement

document collaboration

conflict resolution

suggestions

view count

history

## Non business requirement

latency

consistency

availability - the service should be availlable at all times and show robustness against failure

scalability - a large number of users should be able to use the service at the same time. they can either view the same document or create new documents.

&#x20;

## Estimation

DAU: 80 million

maximum number of users able to edit a document concurrently is 20

size of document: 100KB

Image: 800K, video: 3MB

doc per user per day: 1

Bandwidth

&#x20;

Schema

doc-db: doc\_id(UUID), content

permission: creater\_id,doc\_id, allowed\_id, share\_link

comments: commen\_id, doc\_id, comment\_location, comment\_content

&#x20;

## High level design

## Websockets

使用websockets，而不是HTTP，ws have a long-lasting connection between clients and servers.

They enable full-duplext communication, we can simutaneuosly communicate from client and server.

there is no overhead of HTTP reuquest and response headers.

### microserverses

we choose microservices instead of monolithic because: development is simpler and faster, every service whtihin the architecturei s isolated , therfore, failure of one service doesn't produce a cascading effect.

different component can have different programming languages.

It's easirer to scale and update services individually.

we recommand start-up use monolithic and use microservices after business grow bigger.

### Queue

Use FIFO

### support documentation fomat

markdown

### Support image

use drive/dropbox design

### Database

Master-slave with sharding

### 算法

### CRDT

### OT: Every operation has a \[unified\_timestamp,edit\_loc, edit\_type, edit\_char]. So we can determine which action comes first. This algorithm will maintain document consistency. The OT algoritm should have causality reservation and convergence.

\[siteID, siteCounter, value, positionalIndex] is assigned with each operation. siteId identifies the user.

&#x20;

&#x20;

记录：

根据ddia写合并算法,3 way merge, 2 way merge, 和一个什么数据结构
