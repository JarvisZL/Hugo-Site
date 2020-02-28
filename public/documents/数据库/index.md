# 数据库


# 数据库
<!-- more -->

## **Tips**
- 注意**只有$\cdots$的**的时候，在**关系代数**中是考虑被减数取哪一个关系，而在**关系演算**中是考虑是否要多加一项来保证类似于选过课，买过商品的约束，在**sql**中考虑是否通过类似于where sno in (select sno from SC where...)来保证

### 关于加入成绩的关系代数
- 选修了CS系开设的所有课程且**所修的所有课程成绩合格**
  $$(\pi_{sno,cno}(L) \div \pi_{cno}(\sigma_{dept = 'CS'}(C))) - \pi_{sno}(\sigma_{g < 60}(L))$$
- 选修了CS系开设的所有课程且**这些课程的成绩合格**
  $$\pi_{sno,cno}(\sigma_{g \geq 60}(L))\div \pi_{cno}(\sigma_{dept = 'CS'}(C))$$
- 选修了自己院系开设的所有课程且**这些课程成绩合格**
  $$\pi_{sno}(L) - \pi_{sno}[\pi_{sno,cno}(C \infty S) - \pi_{sno,cno}(\sigma_{g \geq 60}(L))]$$
  <font color=red>像这种对自己院系(城市)的所有什么做了什么的情况通常用减法来表示除法</font>

### 两重所有
- 通过所有销售过所有的商品的供应商购买过商品的客户<br>
    销售过所有商品的供应商：$\pi_{aid,pid}(O) \div \pi_{pid}(P)$
    $$\pi_{cid,aid}(O) \div (\pi_{aid,pid}(O) \div \pi_{pid}(P))$$
- 向NY市所有客户都销售过所有单价大于1的商品的供应商<br>
  此处并非使用两个除法，**需要搞清楚条件是什么!!**,除数应该是NY市的客户和单价大于1的商品两个集合的笛卡尔乘积：$\pi_{cid}(\sigma_{city ='NY'}(C)) \times \pi_{pid}(\sigma_{price > 1}(P))$
  $$\pi_{aid,cid,pid}(O)\div [\pi_{cid}(\sigma_{city ='NY'}(C)) \times \pi_{pid}(\sigma_{price > 1}(P))]$$




