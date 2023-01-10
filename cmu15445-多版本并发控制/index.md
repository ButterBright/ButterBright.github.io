# cmu15445-多版本并发控制


<img src="/cover/mvcc.png"/>

- 读不会阻塞写，写也不会阻塞读。
- 只读事务可以读到连续的快照并用时间戳决定可见性。
- 支持 time-travel 查询。

## MVCC

### SI

### write skew anomaly

<div align="center"><img src="/cmu15445-多版本并发控制/write-skew.png" style="zoom:33%;" /></div>

## design decisions

### concurrency control

- 2pl
- occ
- t/o

### version storage

使用版本链存储不同的版本。

- append-only
  - new to old
    - 不需要遍历查找最新版本，但每次更新 tuple 都要更新索引指针
  - old to new
    - 与 new to old 相反
    <div align="center"><img src="/cmu15445-多版本并发控制/append-only.png" style="zoom:33%;" /></div>
- time-travel
  - main table 存放最新值，time-travel table 存放过去的版本
  <div align="center"><img src="/cmu15445-多版本并发控制/time-travel.png" style="zoom:33%;" /></div>
- delta
  - main table 存放最新值，delta storage segment 存放过去的版本并在 time-travel 的基础上记录 old_value->new_value
  <div align="center"><img src="/cmu15445-多版本并发控制/delta-storage.png" style="zoom:33%;" /></div>

### garbage collection

应该被回收的版本包括：

- 没有活跃的事务对该版本可见
- 该版本由已经终止的事务创建

#### tuple-level

- background vacuuming
  - 周期性地扫描表
  - 使用 bitmap 记录 dirty block
    <div align="center"><img src="/cmu15445-多版本并发控制/background-vacuuming.png" style="zoom:33%;" /></div>
- cooperative cleaning
  - 在遍历版本链地时候进行清理
  - 只适用于 O2N
    <div align="center"><img src="/cmu15445-多版本并发控制/cooperative-cleaning.png" style="zoom:33%;" /></div>

#### transaction-level

在事务终止或提交时为 vacuum worker 提供信息，DBMS 周期性判定什么时候该版本不再可见。

<div align="center"><img src="/cmu15445-多版本并发控制/transaction-level.png" style="zoom:33%;" /></div>

### index management

#### 二级索引

如何对维护二级索引地多版本信息

- 逻辑指针
  - 将二级索引的 tuple id 映射至主键索引或物理地址
  <div align="center"><img src="/cmu15445-多版本并发控制/logical.png" style="zoom:33%;" /></div>
- 物理指针
  - 二级索引指针直接指向版本链
  <div align="center"><img src="/cmu15445-多版本并发控制/physical.png" style="zoom:33%;" /></div>

剩余还有 duplicate key 和 delete 部分被 andy 跳过去懒得记录了 hhh


