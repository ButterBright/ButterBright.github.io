# cmu15445-storage


<img src="/cover/storage.png"/>

## 课程大纲

自底向上依次介绍磁盘管理、缓冲池、访问方法、操作执行和查询计划。

<div align="center"><img src="/cmu15445-storage/outline.png" style="zoom:50%;" /></div>

## 存储介质

非易失的 HDD 和 SSD、内存、CPU 缓存和寄存器。

<div align="center"><img src="/cmu15445-storage/hierarchy.png" style="zoom:50%;" /></div>

## 面向磁盘的数据库系统

### 设计目标

存储超过内存大小的数据，同时兼顾读写性能。

### 为什么不使用 mmap？

- 多线程并发访问导致频繁的缺页中断
- DBMS 想更好地控制内存及磁盘
  - 以正确的顺序写脏页
  - 特定的预读
  - buffer 替换
  - 线程/进程调度

## 数据库存储

### 文件存储

数据库文件和操作系统中的其它文件没什么区别，存储引擎负责管理维护这些文件，以页的形式组织数据。

#### 特点

- 一块数据，包括 tuples、元数据、索引和日志等等
- 每页有一个 id

#### 组织结构

- heap file

  - 链式结构

    <div align="center"><img src="/cmu15445-storage/linkedlist.png" style="zoom:50%;" /></div>

  - 页目录

    <div align="center"><img src="/cmu15445-storage/page-directory.png" style="zoom:50%;" /></div>

### 页格式

<div align="center"><img src="/cmu15445-storage/page-header.png" style="zoom:50%;" /></div>

#### header

header 中包含页的大小、校验和、DBMS 版本、事务的可见性、压缩信息 tuple 存储

#### 存储格式

- 定长顺序存储

  <div align="center"><img src="/cmu15445-storage/fixed-length.png" style="zoom:50%;" /></div>

- slotted pages

  <div align="center"><img src="/cmu15445-storage/slotted-pages.png" style="zoom:50%;" /></div>

- log-structured

  <div align="center"><img src="/cmu15445-storage/log-structured.png" style="zoom:50%;" /></div>

  - 以日志形式组织数据需要定期合并和压缩数据

  <div align="center"><img src="/cmu15445-storage/compact.png" style="zoom:50%;" /></div>

### tuple 格式

<div align="center"><img src="/cmu15445-storage/tuple.png" style="zoom:50%;" /></div>

#### header

- 可见性信息，用于并发控制
- 空值的 bitmap
- 不存放 schema 信息

#### data

<div align="center"><img src="/cmu15445-storage/data.png" style="zoom:50%;" /></div>

- 去规格化

  - 关联的数据存在一起
  - <div align="center"><img src="/cmu15445-storage/denormalize.png" style="zoom:50%;" /></div>

- record id
  - page_id+offset/slot
  - 对上层应用而言没有任何含义

## 数据表示

<div align="center"><img src="/cmu15445-storage/representation.png" style="zoom:50%;" /></div>

### 可变精度数据

- float/real/double
- 使用原生 C/C++类型，IEEE-754
- 相比任意精度更快，但是会有舍入错误

### 固定精度数据

- numeric/decimal

### 大值存储

- overflow page
<div align="center"><img src="/cmu15445-storage/large-value.png" style="zoom:50%;" /></div>
- external file
  - DBMS 不能操作外部文件
  - 没有持久性和事务保证

## 系统目录

- 存放数据库元数据
- 表、列属性、索引、视图、用户权限、统计信息

## 存储模型

### 数据库负载

- OLTP
  - 读/写少量数据
- OLAP
  - 复杂查询，读大量数据
- HTAP
  - OLTP+OLAP

### 数据模型

#### NSM

- 行存储，适用于 OLTP 应用
- 插入、更新和删除以及小范围查询效率高
- 整表查询浪费 io
<div align="center"><img src="/cmu15445-storage/nsm.png" style="zoom:50%;" /></div>

#### DSM

- 列存储，适用于 OLAP 应用
- 高效的 io
- 点查、插入、更新和删除时效率较低
<div align="center"><img src="/cmu15445-storage/dsm.png" style="zoom:50%;" /></div>

#### tuple id

- 定长偏移量
  - 每个属性的值都是定长的
- 嵌入式 id
  - 每个值对应一个 id


