# 第二部分


# Data Cube And OLAP
<!--more-->
## Data warehouse
- 数据仓库是支持管理决策过程的面向主题的，集成的，长期的，稳定的数据集合
- 关系数据库
  - On-line transaction processing(**OLTP**)
- 数据仓库
  - On-line analytical processing(**OLAP**)
- OLAP architecture
![architecture](/images/documents/数据挖掘导论/OLAParchitecture.png)

## Data model for data warehouse
- **多维数据模型(Multidimensional data model)**
  - 维度指的是和那些数据仓库管理者期望相关的视角或实体
- 模型
  - 星型模型: 一个事务表在中间连接一些维度表
    ![star shema](/images/documents/数据挖掘导论/starschema.png)
  - 雪花模型: 在星型模型的基础上可以存在某些维度表向外部连接其他更小的维度表
    ![Snowflake shcema](/images/documents/数据挖掘导论/snowschema.png)

  - 星系模型：由多个星型模型组成，事务表之间由维度表连接起来。
    ![Constellation shcema](/images/documents/数据挖掘导论/constellationschema.png)



## Data Cube
- Data Cube 是由一个个立方体组成的格。其只是一个理论上的东西，并不是实际上数据存储的方式。

### Concept Hierachy(概念层次结构)
- 概念层次结构是一序列的从低层级到高层级的映射组成
- 具体的层次
  - 模式层次：全序和偏序
  - 集分组层次结构：将较大的集合不断拆分成较小的集合

## OLAP在Data Cube上的操作(operations) 
- **roll-up**: 将某一维度提升一个层次结构或者删除某些维度。
  ![roll-up](/images/documents/数据挖掘导论/roll-up.png)
- **drill-down**: 和roll-up操作对立
  ![drill-down](/images/documents/数据挖掘导论/drill-down.png)
- **slice**: 在Data Cube上确定一个维度的值，相当于从立方体上切一片
  ![slice](/images/documents/数据挖掘导论/slice.png)
- **dice**: 在Data Cube上对多个维度的选择，相当于从立方体上切小一小块
  ![dice](/images/documents/数据挖掘导论/dice.png)
- **pivot**: 对Data Cube进行旋转
  ![pivot](/images/documents/数据挖掘导论/pivot.png)

## Data Cube Measures
- 定义：一个data cube measure 是一个能够在data cude空间中每一个点上进行估算的数值函数。
- 分布聚合函数(Distributive aggregate function)：
  - 假设数据分成n份
  - 该函数在每一份上计算出一个聚合值
  - 由该函数在每一份上计算出聚合值派生的结果和由该函数直接作用在整个数据上的结果一致
  - 比如：count(),sum(),min(),max()
- Measures的分类
  - 分布的(Distributive): 通过分布聚合函数得到的，如count(),sum()
  - 代数的(Algebraic): 通过一个代数函数作用在M(M是一个有界的整数)个由分布聚合函数得到的参数上得到，如avg(),min_N(),max_N(),standard_deviation()
  - 整体的(Holistic): 没有一个常数的界来限制用来描述一个子聚合的存储空间。如rank(),median(),mode()

## OLAP 服务架构
- **ROLAP(Relational OLAP)**
  - 使用(拓展)关系数据库来存储数据仓库
  - 有较好的可扩展性
- **MOLAP(Multidimensional OLAP)**
  - 使用基于数组的多维度存储引擎
  - 能很快地索引到 **提前计算好(pre-computed)** 地总计数据
- **HOLAP(Hybrid OLAP)**
  - 同时具备ROLAP的扩展性和MOLAP的计算速度快
- **Specialized SQL servers**
  - 支持在星型模型和雪花模型上的SQL查询

### 预先计算(precompute)
- 提前计算能够保证较快的反映时间和避免冗余的计算
- 预先计算最好的方式：
  - **部分具体化(提前计算部分数据立方体)**，优于无具体化(响应时间长)和全部具体化(存储开销大)

### 有效的计算
- 以ROLAP(基于tuples 和 relational tables)为基础的cube计算
  - 基于一些**优化技巧**，相对较慢
- 以MOLAP(基于multidimensional array)为基础的cube计算
  - 基于**多路数组的聚合**，相对较快

## Multiway array aggregation(多路聚合)
- 定义chunks: 用于存储在内存中小型的立方体
- 通过<font color=red size=3>按照一定顺序访问cube cells来计算多路聚合</font>来最小化cube cells被重复访问的次数

### 例子
- 假设A,B,C三个维度大小为40,400,4000,而chunk的大小为$10 * 100 * 1000$，所以相当于我们在每一维度上都分成了四份
  <img src="/images/documents/数据挖掘导论/多路聚合.png" style="width: 50%">
- 按照序号升序开始访问chunks：
  - 发现当我们访问完1，2，3，4后，$\bm{b_0c_0}$已经计算完成，所以可以将其写入，然后我们访问到5时，就可以复用刚才的缓存空间，所以**BC-plane**需要的缓存是一个chunk的BC面$100 * 1000$
  - 对于**AC-plane**,我们发现只有当访问到13的时候，$\bm{a_0c_0}$才被计算完成，而为了不重复读取2,3,4等的值，所以我们需要开辟缓存来存储，所以一共需要$40 * 1000$
  - 对于**AB-plane**,我们发现只有读到第49个的时候，才能计算完$\bm{a_0b_0}$,所以需要$40 * 400$

## 两种OLAP数据的索引方式：Bitmap Index 和 Join Index
![](/images/documents/数据挖掘导论/bitmapindex.png)
  <img src="/images/documents/数据挖掘导论/joinindex.png" style="width: 50%">

## MetaData
- 元数据在数据仓库中指的是那些定义数据仓库对象的数据
- 分类
  - 潜在数据来源的信息
  - 数据模型的信息
  - 有关事务数据结构和仓库数据结构映射的信息
  - 有关仓库使用的信息

## OLAM架构
<img src="/images/documents/数据挖掘导论/OLAM.png" style="width: 70%">

## AOI(Attribute-oriented induction)
- 最简单的描述性的数据挖掘，可以看作是扩展后的OLAP

### General idea:
- 收集相关数据
- 通过属性删除或者属性泛化来达到泛化数据的目的
- 通过合并相同的，泛化的元组并记录他们各自的计数值来实行聚合
- 关键点是<font color=red size=3>数据泛化</font>

### AOI的大概过程
- Data focusing: 规范化相关数据，初始化工作环境
- Data Generalization：
  - attribute removal：当某一属性有很多不同的数据值时，如果对于这个属性没有聚合操作或者这个属性更高层的概念可以被其他属性所反映，则可以使用attribute removal
  - attribute generalization: 当，某一属性有很多不同的数据值时，并且对于这个属性存在一些聚合操作，那么可以使用attribute generalization
- Presentation

### 怎么控制data generalization
- 属性聚合阈值控制：如果对于一个<font color=red size=3>属性</font>，其不同取值多于该阈值时，会在这个属性上实施属性删除或者属性聚合，一般设置为2-8
- 泛化关系阈值控制：如果对于一个<font color=red size=3>泛化关系</font>上不同元组的数量大于该阈值时，更进一步的泛化将会实施在该关系上

### 定量特征规则
- 一般形式：
  $$\forall X,targetClass(X) \Longrightarrow condition_1(X)[t:w_1]\vee \cdots \vee conditon_n(x)[t:w_n]$$
- $t-weight$: 用于描述<font color=red size=3>当X处于选定的$targetClass$时，$condition_i$满足的概率</font>
- $d-weight$: 用于描述<font color=red size=3>当X满足$condition_i$时，其处于$targetClass$的概率</font>
- 例子：![](/images/documents/数据挖掘导论/finalexample.png)















































