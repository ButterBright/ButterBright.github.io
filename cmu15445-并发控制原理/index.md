# cmu15445-并发控制原理


<img src="/cover/concurrency-control.png"/>

## transactions

### 定义

对于数据库执行的一系列读写操作，是改变数据库的基本单元。

### 正确性标准

- 原子性：所有操作要么全部成功，要么全部失败
- 一致性：事务执行后最终的结果是正确的
- 隔离性：不同事务间是独立执行的
- 持久性：只要事务提交了，结果便可以持久生效

## atomicity

### logging

记录所有终止事务的操作，回退该事务的全部操作

### shadow paging

DBMS 先复制 pages，并且对这些 pages 进行修改，再事务提交后，使这些 pages 对用户可见

## consistency

始终可以保证数据库的约束，从一个正确的状态迁移至另一个正确的状态。

## isolation

不同事务看起来只有它自己在执行，但事实上可能是不同事务交错执行。

两种协议：

- 悲观：不让冲突发生
- 乐观：可以发生冲突，发生冲突后解决冲突

### 如何判断正确性

serial schedule：依次串行执行每个事务

serializable schedule：与 serial schedule 等价的执行安排

如果是 serializable schedule，则该安排是正确的。

### 冲突

#### 冲突类型

- 读写冲突
  - 不可重复读
  <div align="center"><img src="/cmu15445-并发控制原理/unrepeated-reads.png" style="zoom:33%;" /></div>
- 写读冲突
  - 脏读
  <div align="center"><img src="/cmu15445-并发控制原理/dirty-reads.png" style="zoom:33%;" /></div>
- 写写冲突
  - 覆盖未提交的数据
  <div align="center"><img src="/cmu15445-并发控制原理/overwriting.png" style="zoom:33%;" /></div>

#### 冲突串行化

如果可以通过交换连续的不冲突的操作，使该 schedule 与 serial schedule 一致，则这个 schedule 是 conflit serializable 的。

更好的方法：构造优先图，如果没有环，则可以冲突串行化。

<div align="center"><img src="/cmu15445-并发控制原理/precedence-graph.png" style="zoom:33%;" /></div>

有可能不满足冲突串行化，但依然可以返回正确的结果。

<div align="center"><img src="/cmu15445-并发控制原理/inconsistent-analysis.png" style="zoom:33%;" /></div>

#### 视图串行化

定义：

<div align="center"><img src="/cmu15445-并发控制原理/view-serializability-definition.png" style="zoom:33%;" /></div>

例子：

<div align="center"><img src="/cmu15445-并发控制原理/view-serializability.png" style="zoom:33%;" /></div>

### 总结

各种串行化间的关系。

<div align="center"><img src="/cmu15445-并发控制原理/schedules-universe.png" style="zoom:33%;" /></div>

## durability

可以通过 logging 或 shadow paging 实现。


