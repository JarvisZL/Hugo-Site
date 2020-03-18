# 主成分分析(PCA)


## PCA基础
### 常见数据特点
- 数据各个维度之间不是相互独立的
  - 数据的内在维度通常远低于其表面维度。
  - 因此需要降低维度(dimensionality reduction),PCA在降维方法中(可能)是最常用的一种。

### 0阶表示
- 假设我们的数据实际上只有一个点，可能由于噪声的存在我们收集到的数据为$x_1,x_2,\cdots,x_n$.
  - 当噪声不存在时$x_1 = x_2 = \cdots = x_n$
- 不允许使用任何维度，如何最佳表示这个数据点$x$
  - 寻找某个固定的向量$m$使得(即到每个点的距离的平方和最小)
  $$J_1(m) = \mathop{min}\limits_m \sum_{i=1}^n \Vert x_i - m \Vert^2$$
  - 假设$m$是d维的，有n个数据，那么每个数据使用的维度: $\dfrac{d}{n} \mathop{=} \limits_{n \rightarrow 0} 0$
- 最优解(偏导等于0)：
  $$m^* = \mathop{argmin}\limits_m \sum_{i=1}^n \Vert x_i - m \Vert^2 = \dfrac{1}{n}\sum_{i=1}^n x_i = \bar{x}$$

### 一维表示：数据维度间的线性关系
- 数据是$d$维的，但是内在维度可能是$m(\ll d)$维的，PCA用线性来降低维度。 
- $y = w^T x + b\qquad (x\in \reals^d, y\in \reals^m)$
- 怎么选择方向
  - 通过平移使得原点位于数据中心，则消除了b
  - 第一种通过$J_1$推导即最小化误差方差的方法见教材
  - 找方向使得方差尽可能大

#### 形式化: 最大化方差
- 方差是衡量新特征包含信息多少的度量
- 优化目标函数 $J_2(w) = \dfrac{1}{n}\sum_{i=1}^{n} \Vert w^T(x_i-\bar{x})\Vert^2$
- 问题: $J_2$可以无穷大或者无穷小(因为可以w全部乘以一个数)
- 解决办法：加上限制条件$\Vert w\Vert^2 = w^Tw = 1$
  $$\mathop{argmax}\limits_w \dfrac{1}{n}\sum_{i=1}^{n} \Vert w^T(x_i-\bar{x})\Vert^2$$
  $$s.t.\qquad\qquad w^Tw = 1$$

#### 化简
$$\Vert (x_i-\bar{x})^Tw\Vert^2 = ((x_i-\bar{x})^Tw)^T((x_i-\bar{x})^Tw) = w^T(x_i-\bar{x})(x_i-\bar{x})^Tw$$
$$\dfrac{1}{n}\sum_{i=1}^{n} \Vert w^T(x_i-\bar{x})\Vert^2 = w^T\dfrac{1}{n}\sum_{i=1}^n (x_i-\bar{x})(x_i-\bar{x})^T w = w^TCov(x)w$$

#### 优化
- 拉格朗日乘子法：将有约束的优化问题转化为无约束的优化问题
- 拉格朗日函数:
  $$f(w,\lambda) = w^TCov(x)w - \textcolor{red}{\lambda(w^Tw-1)}$$
- 最优的必要条件: $\dfrac{\partial f}{\partial w} = 0,\quad \dfrac{\partial f}{\partial \lambda} = 0$
  - $\dfrac{\partial f}{\partial w} = 2Cov(x)w - 2\lambda w = 0$
  - $Cov(x)w = \lambda w$,说明必要条件是w是$Cov(x)$的一个特征向量
 $$J_2(w) = w^TCov(x)w = w^T\lambda w = \lambda$$
- 即选取最大特征值最大时的特征向量作为w.
- 用w来近似x
  - $\hat{x} \approx \bar{x} + (w^T(x-\bar{x}))w$

#### 重建数据与原来数据的关系
- 当$n > d$(即数据比维数多)时，
  - 我们可以进一步假设$Cov(x)$是一个$\textcolor{red}{可逆矩阵}$，也就是说其此时是$\textcolor{red}{满秩}$的。
  - 则$Cov(x)$有$d$个互相垂直的特征向量$\xi_i$,则有
    $$\forall x,\qquad x \textcolor{red}{=} \bar{x} + \sum_{i=1}^{\textcolor{red}{d}} (w_i^T(x-\bar{x}))w_i$$
  - 此时利用了所有的特征向量，所以没有误差损失。
  - 令$W = [w_1,\cdots,w_d]$，则$x = \bar{x} + WW^T(x-\bar{x})$
- 当$n < d$时，则$Cov(x)$的秩最多为n，所以我们最多有n个相互垂直的特征向量，这就导致会出现一些误差损失。

#### 降维
- 为了降低维度，可以适当的容许一些损失，我们可以通过丢掉特征值小的那一部分来尽可能保证降维后数据的"能量"
- 通常保持90%的能量
  $$\dfrac{\lambda_1 + \lambda_2 + \cdots + \lambda_T }{\lambda_1 + \lambda_2 + \cdots + \lambda_d} > 0.9$$
- 损失：
  - 此时$\hat{W} = [w_1,w_2,\cdots,w_T](d\times T)$
    $$x - \hat{x} = \sum_{i = T+1}^d(w_j^T(x-\bar{x}))w_j \mathop{=}\limits_{(w_j^T(x-\bar{x}))w_j = e_j}  \sum_{i = T+1}^d e_j$$
  - 一些结论 $(1) e_j^Te_k = 0 (j\neq k)\quad (2)E(\Vert x-\hat{x}\Vert^2) = \sum_{i=T+1}^{d}E(\Vert e_j\Vert^2)\quad (3)E(\Vert e_j \Vert^2) = \lambda_j$

### PCA变换步骤
- 训练样本：$x_1,x_2,\cdots,x_n$
- 计算$\bar{x},Cov(x)$
- 求得$Cov(x)$的特征值和特征向量
- 根据特征值选择T,从而确定$\hat{W}$
- 对于数据x，经过变换得到的特征为$\textcolor{red}{y = \hat{W}^T(x-\bar{x})}$
- 重建后的数据$\textcolor{red}{\hat{x} = \bar{x} + \hat{W}y}$
  

## 正态分布和PCA
- 设x满足D维高斯分布
  $$p(x) = (2\pi)^{-\dfrac{D}{2}}\vert \Sigma \vert^{-\dfrac{1}{2}}exp\\{ -\dfrac{1}{2} (x-\mu)^T \Sigma^{-1} (x-\mu)\\}$$
- 对其进行PCA，假设使用全部特征向量则 $y = W^T(x-\mu)$
- x的协方差矩阵$\Sigma = \sum_{i=1}^{d}\lambda_i\xi_i\xi_i^T = W\Lambda W^T$
  - $\Lambda_{ii} = \lambda_i$
- 可以计算$WW^T = W^TW = I$,$Ey = 0,Cov(y) = \Lambda$
- PCA相当于对x数据进行了旋转，是的新的特征y各个维度之间不相关(independent for gaussion)

### 白化变换
- PCA后$y = W^T(x-\mu)$,白化变化(Whitening transform)后$y = (W\Lambda^{-1/2})^T(x -\mu)$
- 此时$y\sim N(0,I)$

### 高斯假设
- PCA变换
  - PCA变换不一定要求x服从高斯分布I
  - 不服从时$E(y) = 0,Cov(y) = \Lambda$但是y不服从高斯分布
  - 不服从时，y的各个维度不相关，但不一定独立
- 白化变换
  - PCA变换不一定要求x服从高斯分布
  - 不服从时$E(y) = 0,Cov(y) = I$但是y不服从高斯分布
  - 不服从时，y的各个维度不相关，但不一定独立

