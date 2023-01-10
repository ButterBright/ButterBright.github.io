# cmu15445-查询计划与优化


<img src="/cover/planning&optimization.png"/>

## architecture

<div align="center"><img src="/cmu15445-查询计划与优化/architecture.png" style="zoom:33%;" /></div>

logical plan vs physical plan(?)

## heuristics optimization

### selection

- 尽早地执行过滤
- 对谓词重新排序，优先执行区分度高的
- 分解谓词并下推

<div align="center"><img src="/cmu15445-查询计划与优化/breaking-predicate.png" style="zoom:33%;" /></div>

### projection

尽早地执行投影操作，需要保留一些字段，比如用来连接地属性。

### others

去除不可能或不必要的谓词，谓词合并等。

## plan cost estimation

### statistics

DBMS 存储了关于表、属性和索引的统计信息。
对于关系 R，DBMS 存储了关系 R 中 tuple 的数量$N_R$和每个属性 A 有多少了不同的值$V(A,R)$。
$SC(A,R)=N_R / V(A,R)$表示平均有多少 tuple 是同一个 value。

### selections(不同谓词的选择性)

$$sel(A=constant) = SC(P) / N_R = 1 / V(A,R)$$
$$sel(A>=a) = (A_{max}-a) / (A_{max}-A_{min})$$
$$sel(not P) = a - sel(P)$$
$$sel(P_1 ⋀ P_2) = sel(P_1) * sel(P_2)$$
$$sel(P_1 ⋁ P_2) = sel(P_1) + sel(P_2) - sel(P_1) * sel(P_2)$$

### assumption

- 数据均匀分布
- 不同的筛选条件是独立的
- 内表中的 key 总能在外表中找到(?)

然而数据的分布不总是均匀的，需要通过改变 bucket 的“宽度”，使得不同 bucket 数据接近均匀分布。

<div align="center"><img src="/cmu15445-连接算法/sort-merge.png" style="zoom:33%;" /></div>

除此之外，不同筛选条件也未必是独立的，而是相互关联的。

### sampling

使用抽样的方式估测不同属性的选择性。当表大幅度改变时，需要更新样本。

## plan enumeration

### single-relation query planning

- 顺序扫描
- 二分查找（对于聚集索引）
- 索引扫描
  从三者中选择最好的 access method 即可，可以进行一定的启发式优化。

### multi-relation query planning

#### System-R

only consider left-deep, usually pipelined, no temp files

<div align="center"><img src="/cmu15445-查询计划与优化/system-r.png" style="zoom:33%;" /></div>

#### dynamic programming

- 枚举 join 顺序
<div align="center"><img src="/cmu15445-查询计划与优化/ordering.png" style="zoom:33%;" /></div>
- 枚举不同算子使用的算法
<div align="center"><img src="/cmu15445-查询计划与优化/algorithm.png" style="zoom:33%;" /></div>
- 枚举 access methods
<div align="center"><img src="/cmu15445-查询计划与优化/access-method.png" style="zoom:33%;" /></div>
<!-- <div align="center"><img src="./cmu15445-查询计划与优化/dp.png" style="zoom:33%;" /></div> -->

### postgres optimizer

连接数量少于 12 时，使用 system-r，否则使用 GEQO。

GEQO:最开始随机生成 query planning，舍弃代价最大的算法，随机反转其余的。

<div align="center"><img src="/cmu15445-查询计划与优化/postgres-optimizer.png" style="zoom:33%;" /></div>

## nested sub-queries

### rewrite

重写 sql，去除嵌套关系。

<div align="center"><img src="/cmu15445-查询计划与优化/rewrite.png" style="zoom:33%;" /></div>

### decompose

先单独执行子查询，将查询结果写入临时文件，执行外层查询时，用临时的结果替换内层查询。

<div align="center"><img src="/cmu15445-查询计划与优化/decompose.png" style="zoom:33%;" /></div>


