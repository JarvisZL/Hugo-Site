# 数学基础


# 模式识别数学基础
<!--more-->

## 线性代数-向量
一般默认为列向量

### 内积
- 定义:

$$\textbf{x}^T\textbf{y}= \sum_{i = 1}^{d}x_iy_i$$

- 显然有 $x^Ty = y^Tx$,可以用于转换

$$(\textbf{x}^T\textbf{y})\textbf{z} = \textbf{z}(\textbf{x}^T\textbf{y}) = \textbf{z}\textbf{x}^T\textbf{y} = \textbf{z}\textbf{y}^T\textbf{x} = (\textbf{z}\textbf{y}^T)\textbf{x}$$

### 范数
- 定义:

$$||\textbf{x}|| = \sqrt{\textbf{x}^T\textbf{x}}$$

- 两个向量之间的距离:
$$||\textbf{x}-\textbf{y}|| = (\textbf{x}-\textbf{y})^T(\textbf{x}-\textbf{y}) = (\textbf{x}^T-\textbf{y}^T)(\textbf{x}-\textbf{y}) = ||\textbf{x}||^2+||\textbf{y}||^2 - 2\textbf{x}^T\textbf{y}$$
- $||\textbf{x}|| = 1$，则**x**为单位向量。
### 角度与投影
- 定义:
  

$$\textbf{x}^T\textbf{y} = ||\textbf{x}||\cdot||\textbf{y}||\cdot \cos{\theta}$$

- 所以$|\textbf{x}^T\textbf{y}| \leq ||\textbf{x}||\cdot||\textbf{y}||$(实际上就是柯西-施瓦茨不等式)
- 特别的，当$\textbf{x}^T\textbf{y} = 0$，则$\textbf{x}\perp\textbf{y}$

- 投影
  - 长度$\Vert\textbf{x}\Vert\cos\theta = \Vert\textbf{x}\Vert\dfrac{\textbf{x}^T\textbf{y}}{\Vert\textbf{x}\Vert\cdot \Vert\textbf{y}\Vert} = \dfrac{\textbf{x}^T\textbf{y}}{\Vert\textbf{y}\Vert}$
  - 方向$\dfrac{\textbf{y}}{\Vert\textbf{y}\Vert}$
  - 投影 $proj_{\textbf{y}} \textbf{x} = \dfrac{\textbf{x}^T \textbf{y}}{\Vert\textbf{y}\Vert^2}\textbf{y}$

## 线性代数-矩阵
- 方阵：矩阵行数等于列数
- 对角阵：方阵中只有对角线非0
- 单位阵：对角线全部为1的对角阵，记为$I_n$或$I$
- 对称阵：$X_{ij} = X_{ji},\forall i,j$
- 转置：$(XY)^T = Y^T\cdot X^T$
### 方阵的行列式和逆矩阵
#### 行列式
- 符号：$|X|$或$det(X)$
- 性质
  - $|X| = |X^T|$
  - $|XY| = |X||Y|$
  - $|\lambda X| = \lambda^n|X|  (X:n\times n)$

#### 逆矩阵
- 定义($X^{-1}$为逆矩阵)
  
    $$XX^{-1} = X^{-1}X = I$$

- 性质
  - X可逆当且仅当$det(X) \neq 0$
  - $(X^{-1})^{-1} = X$
  - $(\lambda X)^{-1} = \dfrac{1}{\lambda}X^{-1}$
  - $(XY)^{-1} = Y^{-1}X^{-1}$
  - $X^{-T} = (X^{-1})^T = (X^T)^{-1}$

### 特征向量和特征值
- 定义：  
  对于一个方阵，如果存在一个非零向量$\textbf{x}$和一个标量$\lambda$满足$A\textbf{x} = \lambda \textbf{x}$，则$\textbf{x},\lambda$分别称为方阵A的特征向量和特征值
- 特征值性质
  - $det(A) = \prod_{i=1}^{n}\lambda_i$
  - $tr(A) = \sum_{i=1}^{n}a_{ii} = \sum_{i=1}^{n}\lambda_i$
- $tr(A)$称为方阵的迹，有性质
  - $tr(XY) = tr(YX)$
  - $tr(X+Y) = tr(X) + tr(Y)$
  - $tr(kX) = ktr(X)$
- 一个方阵的秩$rank(A)$等于其**非零特征值**的数量。

### 实对称矩阵
#### 性质
- 所有特征值都是实数，所有特征向量都是实向量。
- 记特征值为$\lambda_1 \geq \lambda_2 \geq ... \geq \lambda_n$，记特征向量为$\xi_1,...,\xi_n$
  - $\forall i\neq j,\xi_i^T\xi_j = 0$
  - $E = [\xi_1,...,\xi_n]$，则$rank(E) = n$

#### 分解
- 根据上述性质，有$X = \sum_{i = 1}^n \lambda_i\xi_i\xi_i^T$(谱分解)
- 当所有特征向量为单位向量时，方阵E是**正交**方阵，有$X = E\land E^T$,其中$\land$是一个对角阵，$\land_{ii} = \lambda_i$
- **正交矩阵**
  - n阶矩阵A满足$A^TA = I$即$(A^{-1} = A^{T})$,则A为正交阵。
  - 性质
    - A为正交阵，则$A^{-1},A^T$都为正交阵且$det(A) = 1 或 -1$
    - A，B都为正交阵，则$AB$也为正交阵

#### (半)正定矩阵
- 定义：矩阵A是(半)正定矩阵当且仅当对于任意非零实向量$\textbf{x}$,满足
  $$\textbf{x}^TA\textbf{x}(\geq)>0$$  
  记为$A(\succcurlyeq)\succ0$
  $$\textbf{x}^TA\textbf{x} =\sum_{i = 1}^n\sum_{j = 1}^n x_ix_ja_{ij}$$
- 一个实对称矩阵是(半)正定的当且仅当其所有的特征值都是(非负值)正值。
- 一个常用的半正定矩阵是$AA^T或者A^TA$,很容易证明$\textbf{x}^TAA^T\textbf{x} = (A^T\textbf{x})^T(A^T\textbf{x}) = \Vert A^T\textbf{x}\Vert^2 \geq 0$

### 矩阵求导
$$(\dfrac{\partial \textbf{a}}{\partial x})_i = \dfrac{\partial a_i}{\partial x}$$

<div>
$$(\dfrac{\partial A}{\partial x})_{ij} = \dfrac{\partial A_{ij}}{\partial x}$$
</div>

<div>
$$(\dfrac{\partial x}{\partial \textbf{a}})_i = \dfrac{\partial x}{\partial a_i} \quad(\dfrac{\partial x}{\partial A})_{ij} = \dfrac{\partial x}{\partial A_{ij}} \quad (\dfrac{\partial \textbf{a}}{\partial \textbf{x}})_{ij} = \dfrac{\partial a_i}{\partial x_j}$$
</div>

## 概率论与数理统计
### 联合，变换
- 设$x = g(y)$,则
  $$p_Y(y) = p_X(x)\vert\dfrac{dx}{dy}\vert = p_X(g(y)) \vert g^\prime(y)\vert$$
### 均值估计和协方差矩阵
- 训练样本$\textbf{x}_1,...,\textbf{x}_n$
  
  均值的估计:
  $$\bar{\textbf{x}} = \dfrac{1}{n}\sum_{i=1}^n \textbf{x}_i$$
  协方差的估计:
  $$Cov(\textbf{x}) = \dfrac{1}{\textcolor{red}{n}}\sum_{i=1}^n(\textbf{x}_i - \bar{\textbf{x}})(\textbf{x}_i - \bar{\textbf{x}})^T$$
  无偏估计:
  $$Cov(\textbf{x}) = \dfrac{1}{\textcolor{red}{n-1}}\sum_{i=1}^n(\textbf{x}_i - \bar{\textbf{x}})(\textbf{x}_i - \bar{\textbf{x}})^T$$

