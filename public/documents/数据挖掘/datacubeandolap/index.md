# DataCubeAndOLAP


# Data Cube And OLAP
<!--more-->
## Data warehouse
- 数据仓库是支持管理决策过程的面向主题的，集成的，长期的，稳定的数据集合
- 关系数据库
  - On-line transaction processing(**OLTP**)
- 数据仓库
  - On-line analytical processing(**OLAP**)

### Data warehouse architecture
![architecture](/images/documents/数据挖掘导论/OLAParchitecture.png)

### Data model for data warehouse
- **多维数据模型(Multidimensional data model)**
  - 维度指的是和那些数据仓库管理者期望相关的视角或实体
- 模型
  - 星型模型: 一个事务表在中间连接一些维度表
  
    ![star shema](/images/documents/数据挖掘导论/starschema.png)
  - 雪花模型:
    ![Snowflake shcema](/images/documents/数据挖掘导论/snowschema.png)


### Data Cube
- Data Cube 是由一个个立方体组成的格。其只是一个理论上的东西，并不是实际上数据存储的方式。
- 
  


