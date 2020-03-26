# 归一化 FLD 人脸识别


## 特征归一化(normalization)

### 每维度归一
- 很多时候，不同维度需要统一到同样的取值范围。
- 训练集$x_1,\cdots ,x_n$,$x_i = (x_{i1},\cdots,x_{id})$
  - 对于每一维数据为$x_{1j},\cdots, x_{nj}$
  - 取其最小值和最大值$x_{min,j},x_{max,j}$
  - 对于该维度的任何数据$x_{ij} = \dfrac{x_{ij} - x_{min,j}}{x_{max,j} - x_{min,j}}$

### 稀疏数据
- 每一维度归一存在的问题
  - 如果某一维度$max = min$即出现分母为0，此时说明该维度没有什么作用，直接删去。
  - 是否可以统一到别的区间
    - 如[-1,1]: $x_{ij} \leftarrow 2\times (\dfrac{x_{ij} - x_{min,j}}{x_{max,j}-x_{min,j}} - 0.5)$
- 稀疏数据(sparse data):数据中很多纬度值为0，这在归一化中会带来问题。
- 维度值为零有两种可能
  - 该维度无意义(没有采样，对该任务无影响等)，设置为0
  - 该维度的值为0
- 那么对于上面两种情况，在归一化时需要分开考虑
  - 无意义情况：仍然设置为0
  - 值为0的情况：按照正常的归一化

### $l_2$或者$l_1$归一化
- 有的时候各个维度之间取值范围不同是有意义的，但是不同的数据点之间的大小(如向量长度)应保持一致，即对每一个数据点进行归一化
- $l_2$归一化：对于数据$x_i = (x_{i1},\cdots,x_{id})$
  - $x_{ij} \leftarrow \dfrac{x_{ij}}{\Vert x_i \Vert_{l_2}},\qquad \Vert x_i \Vert_{l_2} = \sqrt{x_i^Tx_i}$
- $l_1$归一化：适用于非负特征，对于数据$x_i = (x_{i1},\cdots,x_{id})$
  - 如果数据是直方图，效果最佳
  - $x_{ij} \leftarrow \dfrac{x_{ij}}{\Vert x_i \Vert_{l_1}},\qquad \Vert x_i \Vert_{l_1} = \sum_{j=1}^d \vert x_{ij} \vert$

### 服从高斯分布的归一化
- 有时候可以确信每一个维度都是服从高斯分布
- 对于每一维j,数据为$x_{1j},\cdots,x_{nj}$
  - 计算其均值$\hat{\mu}_j$和方差$\hat{\sigma}_j^2$
  - 对于每一个特征值$x_{ij}\leftarrow \dfrac{x_{ij} - \hat{\mu}_j}{\hat{\sigma}_j}$

### 归一化测试数据
- 在归一化测试数据的时候，我们不能像对待训练集一样，从中取得最大值最小值平均值之类的。
- 需要用训练集归一化时得到的归一化模型(比如参数$x_{max,j}$)，来归一化测试集。

## Fisher线性判别分析(FLD)
- PCA在数据是单个高斯分布是最佳的，其有利于表示数据，但是和分类无关
- 在分类问题中，不同类别的分布$p(\bm{x}\vert y=i)$不能相同，所以FLD用来提取利于分类的特征。
![](/images/documents/模式识别/FLD动机.png)

### 形式化表示(二类问题)
- 希望找到一个投影方向，使得被投影后，两个类别的数据容易被分开。
- 两个类的均值
  $$\bm{\mu_1} = \dfrac{1}{N_1}\sum_{y_i = 1}\bm{x_i}, \bm{\mu_2} = \dfrac{1}{N_2}\sum_{y_i = 2}\bm{x_i}$$
- 投影后均值为$\bm{m_1} = \bm{w}^T\bm{\mu_1},\quad \bm{m_2} = \bm{w}^T\bm{\mu_2}$
- 描述分开的程度-<font color=red size=3>Fisher准则</font>
  - 在要求$\vert \bm{m}_2 - \bm{m}_1 \vert$尽可能大的同时，要求两个在投影后尽可能集中(不分散).
  $$J(\bm{w}) = \dfrac{(\bm{m_2} - \bm{m_1})^2}{s_1^2+s_2^2}$$
- 描述分散程度-<font color=red size=3>散度</font>
  $$s_k^2 = \sum_{y_i = k}(\bm{\mu_i} - \bm{m_k})^2\qquad k = 1,2$$
- 上式称为类内散度。
  $$s_k^2 = \sum_{y_i = k}(\bm{\mu_i} - \bm{m_k})^2 = \sum_{y_i = k}(\bm{w}^T(\bm{x_i} - \bm{\mu_k}))^2 = \bm{w}^T\sum_{y_i = i}(\bm{x_i} - \mu_k)(\bm{x_i} - \mu_k)^T\bm{w}$$
  $$(\bm{m_2} - \bm{m_1})^2 = \bm{w}^T(\bm{\mu_2} - \bm{\mu_1})(\bm{\mu_2} - \bm{\mu_1})^T\bm{w}$$
- 令$S_W = S_1+S_2,\qquad S_B = (\bm{\mu_2} - \bm{\mu_1})(\bm{\mu_2} - \bm{\mu_1})^T$
  $$maxJ(\bm{w}) = \dfrac{\bm{w}^TS_B\bm{w}}{\bm{w}^TS_W\bm{w}},s.t.\bm{w}^T\bm{w} = 1(广义瑞利商)$$

### 求解
- 利用拉格朗日乘子法可以证明最优解时满足$S_B\bm{w} = \lambda S_W\bm{w}$
- 上述问题为广义特征值问题，但在二类问题上有简单解法。
  $$S_B \bm{w} = (\bm{\mu_2} - \bm{\mu_1})( \bm{\mu_2} - \bm{\mu_1})^T \bm{w} \propto (\bm{\mu_2} - \bm{\mu_1})$$
- 因为后面两项可以看成内积为一个实数，我们把实数放入等号右边的$\lambda$中
  $$(\bm{\mu_2} - \bm{\mu_1}) = \lambda^\prime S_W\bm{w}$$
- 变成了一个简单的问题，求解步骤为
  - 计算$\mu_2,\mu_1$
  - 计算$S_W$
  - 计算$\bm{w} = {S_W}^{-1}(\bm{\mu_2} - \bm{\mu_1})$
  - 归一化$\bm{w} \leftarrow \dfrac{\bm{w}}{\Vert \bm{w} \Vert}$

### 不可逆求解
- 如果$S_W$矩阵不可逆，则我们可以使用其广义逆矩阵。
- $S_W$是实对称的，且至少是半正定的(外积性质)
  - $S_W = E\Lambda E^T,\lambda_{ii} \geq 0$
- 定义Moore-Perose伪逆矩阵
  - 如果$\lambda_{ii} > 0$,定义$\lambda_{ii}^+ = \dfrac{1}{\lambda_{ii}}$,否则定义$\lambda_{ii}^+ = 0$
  - 则$S_W^+ = E\Lambda^+E^T$

### 多(C)类问题
- $S_W = \sum_{i=1}^C S_i$,定义总的均值$\bm{\mu} = \dfrac{1}{N} = \sum_x \bm{x}$
- 总的散布矩阵
  $$S_T = \sum_x (\bm{x} - \bm{\mu})(\bm{x} - \bm{\mu})^T$$
  $$S_T = S_W + \sum_{i=1}^C N_i(\bm{\mu_i} - \bm{\mu})(\bm{\mu_i} - \bm{\mu})^T = S_W + S_B$$
- 定义$S_B = \sum_{i=1}^C N_i(\bm{\mu_i} - \bm{\mu})(\bm{\mu_i} - \bm{\mu})^T$
- 广义瑞利商
  $$maxJ(\bm{w}) = \dfrac{\bm{w}^TS_B\bm{w}}{\bm{w}^TS_W\bm{w}}$$
- 拉格朗日乘子法得到$S_B\bm{w_i} = \lambda_i S_W \bm{w_i}$,求解广义特征值问题
- 最多可以得到<font color=red size=3>C-1</font>个有效投影方向
  
