---
title: "基于Wisckey实现kv分离"
date: 2023-10-02T12:36:07+08:00
draft: true
---

## LSM-tree的写放大
LSM-tree在进行compaction过程中需要对多个SST进行merge操作，重写多个kv，从而带来了写放大的问题。

## kv分离的基本思路
Wisckey将SST中的大value写入vlog中，而LSM中只存储对应value在vlog中的位置信息，从而在compaction的过程中减少了需要重写的数据量，有效减小了写放大。对于vlog中的value，则需要定期做垃圾回收，释放已经失效数据所占用的空间。
<div align="center"><img src="/基于Wisckey实现kv分离/compaction-comparison.png" style="zoom:15%;" /></div>

## 数据的写流程
首先将value写入vlog，记录vptr，即value的位置信息，将key和vlog先后写入WAL和memtable中，后台会将memtable中的数据写入SST，并且定期做compaction。
<div align="center"><img src="/基于Wisckey实现kv分离/write-badger.png" style="zoom:20%;" /></div>

## 如何实现垃圾回收
### 触发条件
将每个vlog的垃圾估算值存在DISCARD文件中，当key被更新或者删除时，增加对应vlog的计数器，若计数器达到了阈值，则对该vlog进行垃圾回收。
### 回收过程
维护在vlog中维护head和tail两个指针，为此将head指针向前移动，如果vlog中的value有效，则将该value追加在tail后面，并且更新tail，最终head和tail之间便是要保留的value，head之前的数据全部可以回收。在数据回收之前需要将vptr写入LSM-tree中。
<div align="center"><img src="/基于Wisckey实现kv分离/gc-badger.png" style="zoom:20%;" /></div>

## 数据的读流程
对于Badger来说，由于在进行垃圾回收回写vlog的时候并不会判断该value是否有效，因此可能出现写回的数据反而是过期的情况，因此在读取数据的时候需要穿透LSM-tree的全部level，遍历SST读取有效数据。在获取value的vptr后，根据vptr在vlog中查找实际的value。
由于vlog的数据并非有序的，通过迭代器读取value的过程是随机读，可以采用prefetch的技术来有效利用SSD带宽。
<div align="center"><img src="/基于Wisckey实现kv分离/get-badger.png" style="zoom:20%;" /></div>

## Reference
- [https://www.skyzh.dev/blog/2021-08-07-lsm-kv-separation-overview/](https://www.skyzh.dev/blog/2021-08-07-lsm-kv-separation-overview/)
