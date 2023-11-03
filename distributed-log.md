# Distributed Log

## 设计思路

每台服务器上有client，通过pub/sub方式收集到log aggregator,过滤之后存储到blob，设计data rentention policy和使用log indexer和visualizer

产生警报和error aggregator

&#x20;

IO密集型

每个发送日志的服务器需要自己的标识：使用sequener生成unique identifier，日期可以按着chronological or casual order来简化分析

&#x20;
