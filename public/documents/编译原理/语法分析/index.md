# 语法分析


# 语法分析器
- 基本作用
  - 从词法分析器获得词法单元序列，确认该序列是否可以由语言的文法生成
  - 对于语法错误的程序，报告错误信息
  - 对于语法正确的程序，生成**语法分析树(语法树)**
  ![语法分析器](/images/documents/编译原理/语法分析器.png)
- 分类
  - 自顶向下语法分析器(用于处理LL文法)
    - 从语法分析树的**根部**开始构造语法分析树
  - 自底向上语法分析器(用于处理LR文法)
    - 从语法分析树的叶子开始构造语法分析树
  - 两个第一个L都表示**从左到右，逐个扫描**词法单元

# 上下文无关文法(CFG)

## 组成部分
- **终结符号**：组成串的基本符号(**词法单元**名字)
- **非终结符号**：表示串的集合的语法**变量**
  - 在程序设计语言中通常对应于某个程序构造如$\textit{stmt}$
- **开始符号**：某个被指定的**非终结符号**
  - 对应的串的集合就是文法的语言
- **产生式**：描述将终结符号和非终结符号组成串的方法
  - 形式：头(左)部$\rightarrow$ 体(右)部
  - 头部是一个非终结符号，右边是一个符号串
  - 例如:$\textit{expression} \rightarrow \textit{expression} + \textit{term}$(加法表达式)

## 例子
- 简单算术表达式文法
  - 终结符号: $id,+,-,*,/,(,)$
  - 非终结符号：$\textit{expression}, \textit{term}, \textit{factor}$
  - 开始符号：$\textit{expression}$
  - 产生式集合
    - $\textit{expression} \rightarrow \textit{expression} + \textit{term}$
    - $\textit{expression} \rightarrow \textit{expression} - \textit{term}$
    - $\textit{expression} \rightarrow \textit{term}$
    - $\textit{term} \rightarrow \textit{term} * \textit{factor}$
    - $\textit{term} \rightarrow \textit{term} / \textit{factor}$
    - $\textit{term} \rightarrow \textit{factor}$
    - $\textit{factor} \rightarrow (\textit{expression})$
    - $\textit{factor} \rightarrow id$
  - 简单表示方法
    - $E\rightarrow E + T | E - T | T$
    - $T\rightarrow T*F | T / F | F$
    - $F\rightarrow (E) | id$
    - 上面的$|$是**元符号**

## 推导
- 将待处理的串的某个**非终结符号**替换为这个非终结符号的某个产生式的**体**(用右边替换左边)
- 从**开始符号**出发，不断替换，最终可以得到不同的 **句型**
- 正式定义
  - 如果$A\rightarrow \gamma$是一个产生式，那么$\alpha A \beta \Longrightarrow \alpha \gamma \beta$
  - **最左(右)推导**：$\alpha(\beta)$中**不包含**非终结符号
    
    - 符号$\mathop{\Longrightarrow}\limits_{lm}^{*},\mathop{\Longrightarrow}\limits_{rm}^{*}$
  - 经过零步或多步推导出:$\mathop{\Longrightarrow}\limits^{*}$
    - 对于任何串$\alpha \mathop{\Longrightarrow}\limits^{*} \alpha$
    <div>
    - 如果$\alpha \mathop{\Longrightarrow}\limits^{*} \beta$且$\beta \Longrightarrow \gamma$则$\alpha \mathop{\Longrightarrow}\limits^{*} \gamma$
    </div>
  - 经过一步或者多步推导出:$\mathop{\Longrightarrow}\limits^{+}$
    
    - $\alpha \mathop{\Longrightarrow}\limits^{+} \beta$等价于$\alpha \mathop{\Longrightarrow}\limits^{*} \beta$且$\alpha \neq \beta$

## 句型/句子/语言
- **句型(Sentential form)**
  - 如果$S\mathop{\Longrightarrow}\limits^{*} \alpha$，那么$\alpha$就是文法S的句型
  - 可能既包含非终结符号又包含中介符号，也可以是空串
- **句子(Sentence)**
  - 文法的句子就是不包含非终结符号的句型
- **语言**
  - 文法G的语言就是G的句子的集合，记为L(G)
  - $\omega$在L(G)中当且仅当$\omega$是G的句子
  - 如何验证文法G确定的语言L
    - 证明G生成的每个串都在L中
    - 证明L中的每个串都能被G生成

## 语法分析树
- 推导的图形表示形式
  - **根结点**的标号是文法的<font color=red size=3>开始符号</font>
  - 每个**叶子结点**的标号是非终结符号，终结符号或者$\epsilon$
  - 每个**内部结点**的标号都是非终结符号
  - 每个内部节点表示某个产生式的一次应用
- 树的叶子组成的<font color=red size=3>序列</font>的根的文法符号的一个句型
- 一颗语法树可对应多个推到序列
  - 但只有<font color=red size=3>唯一的最左推导及最有推导</font>

### 二义性(Ambiguity)
- 定义：如果一个文法可以为某个句子生成<font color=red size=3>多棵</font>语法树，这个**文法**就是二义的
- 例子: ![二义性](/images/documents/编译原理/二义性.png)
- 程序设计语言的文法通常是无二义的

# 设计文法
- 在进行高效的语法分析前，需要对文法做以下处理：
  - 消除<font color=red size=3>二义性</font>
  - 消除<font color=red size=3>左递归</font>
    - **左递归**：文法中一个非终结符号A使得对某个串，存在一个推导$\textit{A} \mathop{\Longrightarrow}\limits^{+} \textit{A}\alpha$，则称这个文法是左递归的
    - 提取<font color=red size=3>左公因子</font>

## 二义性的消除
- 一些二义性文法可以被改成等价的无二义性的文法

## 左递归的消除
- 定义：如果一个文法中有非终结符号$A$使得$A \mathop{\Longrightarrow}\limits^{+} A\alpha$,那么这个文法就是左递归的。
- **立即左递归(规则左递归)**:文法中存在一个形如$A \rightarrow A\alpha$的产生式
- 自顶向下的语法分析需要消除左递归。

### 立即左递归的消除
- 存在立即左递归 
  $$A \rightarrow A\alpha_1 ~ \vert ~ \cdots ~ \vert ~ A\alpha_m ~ \vert ~ \beta_1 ~ \vert ~ \cdots ~ \vert ~ \beta_n$$
  可替换为
  $$A\rightarrow \beta_1A^\prime ~ \vert ~ \cdots ~ \vert ~  \beta_nA^\prime$$
  $$A^\prime \rightarrow \alpha_1A^\prime ~ \vert ~ \cdots \vert ~ \alpha_m A^\prime ~\vert ~ \epsilon  $$

### 消除多步左递归
- 输入：没有环和$\epsilon$产生式的文法G
- 输出：等价的<font color=red size=3>没有左递归</font>的文法
- 步骤：将文法的非终结符号排序为$A_1,\cdots,A_n$
  ![消除左递归](/images/documents/编译原理/消除左递归.png)

## 预分析法
- 每次为最左边的非终结符号选择适当的产生式
  - 通过查看<font color=red size=3>下一个输入符号</font>来选择产生式，例如：$E\rightarrow +EE~\vert~-EE~\vert~id~$，输入$+~id~-id~id$
  - <font color=blue size=3>当有多个可能的选择时预分析法则无能为力</font>,例如当两个产生式拥有<font color=red size=3>相同的前缀</font>的时候，例子：$S \rightarrow \textbf{if}~ E~ \textbf{else}~ E~ \textbf{then}~ E~\vert~ \textbf{if}~ E~\textbf{else}~E$
  - 可以通过提取<font color=red size=3>公因式</font>的方法来解决

### 提取公因式
- 输入：文法G
- 输入：等价的提取了<font color=red size=3>左公因子</font>的文法
- 方法：对于每个非终结符号A，找出它的两个或者多个可选产生式之间的<font color=red size=3>最长公共前缀</font>
  - $A\rightarrow \alpha\beta_1~\vert~\cdots~\vert~\alpha\beta_n~\vert~\gamma$
  - $A\rightarrow \alpha A^\prime~\vert~\gamma,\qquad A^\prime \rightarrow \beta_1~\vert~\cdots~\vert~\beta_n$

### 自顶向下的语法分析
- 为输入串构建语法树
  - 从语法树的$\textcolor{blue}{根节点}$开始，依照<font color=red size=3>先根次序</font>，深度优先创建各个结点
  - 对应<font color=red size=3>最左推导</font>
- 基本步骤：确定对句型中<font color=red size=3>最左边</font>的非终结符号应用哪个产生式，然后用产生式与输入符号进行匹配

### 递归下降分析技术
- 每个非终结符号对应一个过程，每个过程扫描该非终结符号对应的结构
  ![递归下降](/images/documents/编译原理/递归下降.png)

### 递归下降分析技术的回溯
- 没有足够多的信息来确定唯一可以选择的产生式，那么在分析过程中就产生<font color=red size=3>回溯</font>
  - 算法在第<font color=red size=3>7</font>行和第<font color=red size=3>4</font>行都可能出现错误或者返回错误，此时并不能说明输入串不是句子，而有可能是选错了产生式，所以我们可以**返回第一行选择下一个可能的产生式**重复

### FIRST和FOLLOW
- 在自顶向下的分析技术中，使用向前看几个符号来确定产生式（通常只看一个符号)
- 当前句型是$x\small \textcolor{red}{A}\beta$,而输入是$x\small \textcolor{red}{a} \cdots$,那么选择产生式的<font color=blue size=3>必要条件</font>是下列之一：
  - $\alpha \mathop{\Longrightarrow} \limits^{*} \small \textcolor{red}{a} \cdots$
  - $\alpha \mathop{\Longrightarrow} \limits^{*} \small \textcolor{red}{\epsilon}$,且$\beta$以$\small \textcolor{red}{a}$即在某个句型中a跟在A之后
  - 如果按照这两个条件选择时能够保证<font color=red size=3>唯一性</font>，那么我们可以避免回溯

#### FIRST定义
- 定义$FIRST(\alpha)$: 如果从$\alpha$推导得到串的<font color=red size=3>首符</font>的集合，如果$\alpha \mathop{\Longrightarrow}\limits^{*} \epsilon$,那么$\epsilon$也在$FIRST(\alpha)$中
- 意义：
  - A的产生式$A \rightarrow \alpha~\vert~\beta$,且$FIRST(\alpha)$和$FIRST(\beta)$<font color=red size=3>不相交</font>,那么对于下一个输入符号a,如果$a \in FIRST(\alpha)$,则选择$A \rightarrow \alpha$，$a\in FIRST(\beta)$同理
  
#### FIRST的计算方法
- 计算$FISRST(X)$
  - X是终结符号，那么加入X
  - X是非终结符号，且$X \rightarrow Y_1Y_2\cdots Y_k$是产生式
    - 如果a在$FIRST(Y_i)$中，且$\epsilon$在$FIRST(Y_1)\cdots FIRST(Y_{i-1})$中，那么加入a
    - 如果$\epsilon$在$FIRST(Y_1)\cdots FIRST(Y_k)$中，那么加入$\epsilon$
- 计算$FIRST(X_1X_2\cdots X_n)$
  - 加入$FIRST(X_1)$中所有<font color=red size=3>非$\epsilon$</font>符号
  - 若$\epsilon$在$FIRST(X_1)$中，加入$FIRST(X_2)$中所有<font color=red size=3>非$\epsilon$</font>符号，以此类推
  - 如果$\epsilon$在所有$FIRST(X_i)$中，则加入$\epsilon$

#### FOLLOW定义
- 定义$FOLLOW(A)$: 在某些句型中紧跟在A<font color=red size=3>右边</font>的<font color=red size=3>终结符号集合</font>，例如$S \rightarrow \alpha A \small \textcolor{red}{a} \beta$,终结符号$a\in FOLLOW(A)$
- 意义：
  - 如果$A\rightarrow \alpha$当$\alpha \rightarrow(\Longrightarrow) \epsilon$时，$FOLLOW(A)$可以帮助我们选择合适的产生式
  - 例如：$A\rightarrow \alpha$，而且$b \in FOLLOW(A)$,如果$\alpha \Longrightarrow \epsilon$,而且当前输入符号是$b$，那么可以选择$A\rightarrow \alpha$

#### FOLLOW的计算方法
- 算法：
  - 将**右端结束标记<font color=red size=3>\$</font>**加入$FOLLOW(S)$中
  - 按照下面规则<font color=red size=3>迭代</font>,直到所有的FOLLOW集合不在变化
    - 如果存在$A\rightarrow \alpha B \beta$, 那么$FIRST(\beta)$中所有<font color=red size=3>非$\epsilon$</font>的符号都加入$FOLLOW(B)$中
    - 如果存在$A \rightarrow \alpha B$或者$A\rightarrow \alpha B \beta$且$FIRST(\beta)$中包含$\epsilon$,那么<font color=red size=3>$FOLLOW(A)$</font>中所有符号都加入$FOLLOW(B)$中

## LL(1)文法
- 定义：对文法任意的两个产生式$A \rightarrow \alpha~\vert~\beta$
  - 不存在终结符号a使得$\alpha,\beta$都可以推出以a开头的串
  - $\alpha,\beta$中至多可以有一个推导出空串，那么如果$\beta$可以推出空串，$\alpha$就不能推出以$FOLLOW(A)$中任何终结符开头的串，否则就会使得$\alpha$也可以推出空串
- 上面的定义等价于： 
  - $FIRST(\alpha) \cap FIRST(\beta) = \emptyset$
  - 如果$\epsilon in FIRST(\beta)$,那么$FIRST(\alpha) \cap FOLLOW(A) = \emptyset$

## 预测分析表构造算法
- 输入：文法G
- 输出：预测分析表M
- 方法
  - 对于文法G中每个产生式$A\rightarrow \alpha$
    - 对于$FIRST(\alpha)$中的每个终结符$\small \textcolor{red}{a}$,将$A\rightarrow \alpha$加入到$M[A,\small \textcolor{red}{a}]$中，如果$\epsilon \in FIRST(\alpha)$,那么对于$FOLLOW(A)$中的每个符号$\small \textcolor{red}{b}$将$A\rightarrow \alpha$加入到$M[A,\small \textcolor{red}{b}]$中
    - 在空白条目中填入<font color=red size=3>error</font>
  - 例子：![](/images/documents/编译原理/预分析表.png)

