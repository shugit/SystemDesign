# Top K/Sharded Counter

## 相关LC题

merge n sorted list [https://leetcode.com/problems/merge-k-sorted-lists/description/](https://leetcode.com/problems/merge-k-sorted-lists/description/)

top k [https://leetcode.com/problems/top-k-frequent-elements/](https://leetcode.com/problems/top-k-frequent-elements/)

&#x20;

[https://www.1point3acres.com/bbs/thread-294114-1-1.html](https://www.1point3acres.com/bbs/thread-294114-1-1.html)

[https://www.1point3acres.com/bbs/thread-909434-1-1.html](https://www.1point3acres.com/bbs/thread-909434-1-1.html)

&#x20;

ref:

[https://soulmachine.gitbooks.io/system-design/content/cn/bigdata/heavy-hitters.html](https://soulmachine.gitbooks.io/system-design/content/cn/bigdata/heavy-hitters.html)

[https://xunhuanfengliuxiang.gitbooks.io/system-desing/content/top-k-in-5-mins-10-mins-1-hour-24-hours.html](https://xunhuanfengliuxiang.gitbooks.io/system-desing/content/top-k-in-5-mins-10-mins-1-hour-24-hours.html)

[https://github.com/water685/system-design/blob/master/linkedin/topk.md](https://github.com/water685/system-design/blob/master/linkedin/topk.md)

[https://www.raychase.net/6275](https://www.raychase.net/6275)

[https://www.raychase.net/6269](https://www.raychase.net/6269)

[https://ravisystemdesign.substack.com/p/interview-preparation-design-a-system](https://ravisystemdesign.substack.com/p/interview-preparation-design-a-system)

## &#x20;

## 我的设计

{% embed url="https://whimsical.com/top-k-59mCTTXyH9EvAKDvZez3NZ" %}





## Business req

Top K API (k, startTime, endTime)

如果是sharded counter的话则有 create counter, write counter, read counter

&#x20;

&#x20;

## non-func req

scalable

highly available

highly performant

accurate

&#x20;

## 实现

分为fast path和slow path

sort hash table o(nlogn)，或者heap nlog(k) k是heap大小
