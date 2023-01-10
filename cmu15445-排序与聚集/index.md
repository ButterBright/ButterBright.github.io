# cmu15445-排序与聚集


<img src="/cover/sorting&aggregations.png"/>

## query plan

- 算子按照树形结构组织
- 数据从叶子结点流向根结点
- 根节点输出最终的查询结果
- 不能假设查询结果都能放在内存当中，需要借助 buffer pool，并尽可能使用顺序 io

<div align="center"><img src="/cmu15445-排序与聚集/query-plan.png" style="zoom:33%;" /></div>

## external merge sort

### 为什么要排序？

- 加速去重(distinct)
- 加速聚集(group by)
- 构建 b+树索引更加高效

### 2-way

buffer pool size = 3 即可，因为每次只能比较两个 pages

<div align="center"><img src="/cmu15445-排序与聚集/2-way.png" style="zoom:50%;" /></div>

### double buffering 优化

- 提前将接下来要排序或合并的 page 加载到另一个 buffer pool 当中
- 可以减少 io 等待时间

### io 次数

<div align="center"><img src="/cmu15445-排序与聚集/io-cost.png" style="zoom:33%;" /></div>

## using b+ trees for sorting

只需要顺序扫描叶子结点

- 聚集索引
<div align="center"><img src="/cmu15445-排序与聚集/clustered-index.png" style="zoom:33%;" /></div>
- 非聚集索引
  - 顺序扫描非聚集索引会导致随机 io，效率很低
  <div align="center"><img src="/cmu15445-排序与聚集/unclustered-index.png" style="zoom:33%;" /></div>

## aggregations

### 通过排序聚集

经过排序后的数据一定是按属性聚集的。

<div align="center"><img src="/cmu15445-排序与聚集/sorting-aggregation.png" style="zoom:33%;" /></div>

### 通过哈希聚集

如何不需要数据有序，仅需要聚集，则哈希是个更好的方案，复杂度更低

#### 步骤

- partition
  - 根据哈希值将 tuple 放到对应的 bucket 中
  - 如果内存满了则写磁盘
- rehash
  - 为每个分区构建单独的哈希表，并进行聚集

#### partition

<div align="center"><img src="/cmu15445-排序与聚集/partition.png" style="zoom:33%;" /></div>

#### rehash

对于每一个分区：

- 使用一个新的哈希函数构建哈希表
- 遍历每一个 bucket，将聚集匹配项
- 这里的前提假设是每一个分区都能存在内存当中

<div align="center"><img src="/cmu15445-排序与聚集/rehash.png" style="zoom:33%;" /></div>

#### hashing summarization

以 k-v 对的形式存储，value 由聚集函数决定，如下图所示。

<div align="center"><img src="/cmu15445-排序与聚集/hashing-summarization.png" style="zoom:50%;" /></div>


