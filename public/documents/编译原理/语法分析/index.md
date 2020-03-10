# 语法分析

# 语法分析

## 语法分析器
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

## 上下文无关文法(CFG)

### 组成部分
- **终结符号**：组成串的基本符号(**词法单元**名字)
- **非终结符号**：表示串的集合的语法**变量**
  - 在程序设计语言中通常对应于某个程序构造如$\textit{stmt}$
- **开始符号**：某个被指定的**非终结符号**
  - 对应的串的集合就是文法的语言
- **产生式**：描述将终结符号和非终结符号组成串的方法
  - 形式：头(左)部$\rightarrow$ 体(右)部
  - 头部是一个非终结符号，右边是一个符号串
  - 例如:$\textit{expression} \rightarrow \textit{expression} + \textit{term}$(加法表达式)

### 例子
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

### 推导
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

### 句型/句子/语言
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

### 语法分析树
- 推导的图形表示形式
  - **根结点**的标号是文法的<font color=red size=3>开始符号</font>
  - 每个**叶子结点**的标号是非终结符号，终结符号或者$\epsilon$
  - 每个**内部结点**的标号都是非终结符号
  - 每个内部节点表示某个产生式的一次应用
- 树的叶子组成的<font color=red size=3>序列</font>的根的文法符号的一个句型
- 一颗语法树可对应多个推到序列
  - 但只有<font color=red size=3>唯一的最左推导及最有推导</font>

#### 二义性(Ambiguity)
- 定义：如果一个文法可以为某个句子生成<font color=red size=3>多棵</font>语法树，这个**文法**就是二义的
- 例子: ![二义性](/images/documents/编译原理/二义性.png)
- 程序设计语言的文法通常是无二义的

## 设计文法
- 在进行高效的语法分析前，需要对文法做以下处理：
  - 消除<font color=red size=3>二义性</font>
  - 消除<font color=red size=3>左递归</font>
    - **左递归**：文法中一个非终结符号A使得对某个串，存在一个推导$\textit{A} \mathop{\Longrightarrow}\limits^{+} \textit{A}\alpha$，则称这个文法是左递归的
    - 提取<font color=red size=3>左公因子</font>

### 二义性的消除
- 一些二义性文法可以被改成等价的无二义性的文法

### 左递归的消除
- 定义：如果一个文法中有非终结符号$A$使得$A \mathop{\Longrightarrow}\limits^{+} A\alpha$,那么这个文法就是左递归的。
- **立即左递归(规则左递归)**:文法中存在一个形如$A \rightarrow A\alpha$的产生式
- 自顶向下的语法分析需要消除左递归。

#### 立即左递归的消除
- 存在立即左递归 
  $$A \rightarrow A\alpha_1 ~ \vert ~ \cdots ~ \vert ~ A\alpha_m ~ \vert ~ \beta_1 ~ \vert ~ \cdots ~ \vert ~ \beta_n$$
  可替换为
  $$A\rightarrow \beta_1A^\prime ~ \vert ~ \cdots ~ \vert ~  \beta_nA^\prime$$
  $$A^\prime \rightarrow \alpha_1A^\prime ~ \vert ~ \cdots \vert ~ \alpha_m A^\prime ~\vert ~ \epsilon  $$

#### 消除多步左递归
- 输入：没有环和$\epsilon$产生式的文法G
- 输出：等价的<font color=red size=3>没有左递归</font>的文法
- 步骤：将文法的非终结符号排序为$A_1,\cdots,A_n$
  ![消除左递归](/images/documents/编译原理/消除左递归.png)

### 预分析法
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

### LL(1)文法
- 定义：对文法任意的两个产生式$A \rightarrow \alpha~\vert~\beta$
  - 不存在终结符号a使得$\alpha,\beta$都可以推出以a开头的串
  - $\alpha,\beta$中至多可以有一个推导出空串，那么如果$\beta$可以推出空串，$\alpha$就不能推出以$FOLLOW(A)$中任何终结符开头的串，否则就会使得$\alpha$也可以推出空串
- 上面的定义等价于： 
  - $FIRST(\alpha) \cap FIRST(\beta) = \emptyset$
  - 如果$\epsilon in FIRST(\beta)$,那么$FIRST(\alpha) \cap FOLLOW(A) = \emptyset$

### 预测分析表构造算法
- 输入：文法G
- 输出：预测分析表M
- 方法
  - 对于文法G中每个产生式$A\rightarrow \alpha$
    - 对于$FIRST(\alpha)$中的每个终结符$\small \textcolor{red}{a}$,将$A\rightarrow \alpha$加入到$M[A,\small \textcolor{red}{a}]$中，如果$\epsilon \in FIRST(\alpha)$,那么对于$FOLLOW(A)$中的每个符号$\small \textcolor{red}{b}$将$A\rightarrow \alpha$加入到$M[A,\small \textcolor{red}{b}]$中
    - 在空白条目中填入<font color=red size=3>error</font>
  - 例子：![](/images/documents/编译原理/预分析表.png)

### LL(1)文法的递归下降分析
- 有了预测分析表后，递归下降不再需要回溯，利用预测分析表直接唯一确定产生式

### 非递归预测分析
- 在自顶向下分析过程中：
  - 匹配句型左边所有的非终结符号
  - 对于最左边的非终结符号，选择适当的产生式展开
  - 已经匹配成功的终结符号不会被考虑，因此只需要记住余下的部分
  - 可以使用一个<font color=red size=3>栈</font>来存放
- 分析时处理过程：
  - 初始化时，栈中仅包含开始符号S(和$\$$)
  - 如果栈顶元素是终结符号，那么进行匹配
  - 如果栈顶是非终结符，
    - 使用预测分析表选择适当的产生式
    - 在栈顶用产生式右部替换产生式左部
- 分析表驱动的预测分析器
  - ![](/images/documents/编译原理/分析表驱动预测分析器.png)
  - 预测分析程序使用$M[X,a]$来扩展X，将产生式右部按<font color=red size=3>倒序</font>压入栈中

### 预测分析算法
- 输入：串$\omega$，预测分析表M
- 输出：如果$\omega$是句子，输出其最左推导，否则报错
- (1) 初始化：输入缓冲区$\omega\$$,栈中为$S\$$，ip指向$\omega$的第一个符号
- (2) 令X = 栈顶元素，ip指向输入符号a<br>
  if (X == a) X出栈，ip向前移动(与终结符号匹配成功)<br>
  else if (X是终结符号) error()(匹配失败)<br>
  else if (M[X,a]是报错条目) error(匹配失败)<br>
  else if (M[X,a] = $X \rightarrow Y_1\cdots Y_k$){<br>
    输出产生式$X \rightarrow Y_1\cdots Y_k$<br>
    弹出栈顶符号X，并将$Y_k,\cdots,Y_1$压入栈中
  }
- (3) 不断执行第二步，直到报错或者栈为空


### 自底向上的语法分析
- 从叶子结点开始生成语法树，通用框架：<font color=red size=3>移入-归约</font>
- 简单LR技术(SLR),LR技术(LR)
  - 其中的R表示最右推导和逆序

### 归约
- 自底向上的语法分析可以看成是从串w归约为文法开始符号S的过程
- 归约步骤
  - 一个与某产生式体相匹配的特定子串被替换为该产生式头部的非终结符号

### 句柄
- 对输入从左到右扫描，并进行自底向上的语法分析，实际可以反向构造一个最右推导
- 定义：$S\mathop{\Longrightarrow} \limits_{rm}^* \alpha \textcolor{red}{A}w \mathop{\Longrightarrow}\limits_{rm} \alpha \textcolor{red}{B} w$,那么紧跟在$\alpha$之后的$\beta$是$A\rightarrow \beta$的一个句柄

### 移入-归约分析技术
- 使用栈保存一如的符号
- 栈中符号和待扫描符号组成一个最右句型
- 开始：栈中只包含$\$$，而输入为$w\$$
- 结束：栈中为$S\$$,输入为$\$$
- 分析过程：不断移入，识别句柄则归约

### 主要分析动作
- 移入：将下一个符号移入到栈顶
- 归约：将句柄归约为相应的非终结符号
  - 弹出句柄，压入非终结符号
- 接受：宣布分析过程完成
- 报错：发现语法错误，调用恢复子程序

### 冲突
- 移入-归约冲突：此时栈中已经可以归约，但读入k个输入后又可以归约，比如if then else 和 if then
- 归约-归约冲突：不知道应该用哪一个产生式来归约

### LR语法分析技术
- LR(k)：最左扫描，最右推导逆序，最多向前看k个符号，只考虑k=0或1的情况

### LR(0)项
- 项：文法的一个产生式加上在其中某处的一个点
  - $A\rightarrow \cdot XYZ, A \rightarrow X\cdot YZ,\cdots$
  - $A\rightarrow \epsilon$对应一个项$A\rightarrow \cdot$
- 直观含义：$A\rightarrow \alpha\cdot \beta$表示已经扫描/归约到了$\alpha$，并期望接下来的输入中经过扫描/归约到$\beta$然后归约到A

### LR(0)项集规范族的构造
- 相关定义
  - 增广文法
  - 项集闭包CLOSURE
  - GOTO函数
- 增广文法
  - G的增广文法$G^\prime$是在G中添加新开始符号$S^\prime$，并加入产生式$S^\prime \rightarrow S$而得到的
  - 显然两个文法接受相同的语言
- 项集闭包CLOSURE
  - 如果I是文法G的一个项集，CLOSURE(I)就是根据下列两条规则从I构造得到的项集
    - 将I中各项加入CLOSURE(I)
    - 如果$A \rightarrow \alpha.B\beta$在CLOSURE(I)中，而$B\rightarrow \gamma$是一个产生式，且项$B\rightarrow \cdot \gamma$不在CLOSURE(I)中，就将该项加入其中，不断增加直到没有新项可以加入
  - 构造算法![](/images/documents/编译原理/CLOSURE(I).png)
- GOTO函数
  - I是一个项集，X是一个文法符号，则GOTO(I,X)定义为I中所有形如项$[A \rightarrow \alpha\cdot X\beta]$所对应的项$[A\rightarrow \alpha X\cdot \beta]$的集合的闭包
  - ![](/images/documents/编译原理/GOTO.png)

### 求LR(0)项集规范族的算法
- 从初始项集开始，不断计算各种可能的后继，知道生成所有项集。
![](/images/documents/编译原理/LR(0)项集规范族算法.png)

### LR(0)自动机的构造
- 构造方法
  - 基于规范LR(0)项集族可以构造LR(0)自动机
  - 规范LR(0)项集族种的每个项集对应于LR(0)自动机的一个状态
  - 状态转换：如果$GOTO(I,X) = J$,则从I到J有一个标号X的转换
  - 开始状态：$CLOSURE(\{S^\prime \rightarrow \cdot S\})$

### LR(0)自动机的作用
- 假设文法符号串$\gamma$使得自动机从开始状态运行到状态J
  - 如果J中存在项$A \rightarrow \alpha \cdot$,那么
    - 在$\gamma$之后添加一些终结符号可以得到一个最右句型。
    - $\alpha$是$\gamma$的后缀，且是该句型的句柄
    - 表示可能找到了当前最右句型的句柄。
  - 如果J中存在项$B \rightarrow \alpha \cdot X\beta$，那么
    - 在$\gamma$之后添加$X\beta$和一些终结符号可以得到一个最右句型
    - 该句型中$\alpha X \beta$是句柄，但还没有找到。

### LR语法分析器的结构
![](/images/documents/编译原理/LR语法分析器结构.png)
- 两个部分：ACTION和GOTO
- ACTION表项有两个参数：状态i，终结符a
  - 移入j: j是新状态，把j压栈(同时移入a)
  - 归约$A\rightarrow \beta$: 把栈顶的$\beta$归约成A，并根据GOTO表项压入新状态
  - 接受：接受输入，完成分析
  - 报错：在输入中发现语法错误
- GOTO表项
  - 如果$GOTO[I_i,A] = I_j(GOTO函数)$，那么$GOTO[i,A] = j(GOTO表项)$

### LR语法分析器格局
- LR语法分析器的格局包含了栈中内容和余下输入$(s_0s_1\cdots s_m,a_ia_{i+1}\cdots a_n\$)$
  - 第一个分量是栈中内容(右侧是栈顶)
  - 第二个分量是余下输入
- 每一个状态对应一个文法符号($s_0$除外)
  - $X_i对应s_i$,则$X_1X_2\cdots X_ma_i\cdots a_n$是一个最右句型
- 对于格局$(s_0s_1\cdots s_m,a_ia_{i+1}\cdots a_n\$)$，LR语法分析器查找$ACTION[s_m,a_i]$
  - 移入s：将状态s移入栈中，得到新格局$(s_0s_1\cdots s_ms,a_{i+1}\cdots a_n\$)$
  - 归约$A\rightarrow \beta$：将栈顶$\beta$归约为A，压入状态s，得到新格局$(s_0s_1\cdots s_{m-r}s,a_{i+1}\cdots a_n\$)$，其中r是$\beta$的长度,状态$s = GOTO[s_{m-r},A]$
    - 接受：分析过程完成
    - 报错：发现语法错误，进行错误恢复。

### LR语法分析算法
- 输入：文法G的LR语法分析表，输入串w
- 输出：如果w在L(G)中，则输出自底向上语法分析的归约步骤，否则输出错误指示
  ![](/images/documents/编译原理/LR语法分析算法.png)

### SLR语法分析表的构造
- 以LR(0)自动机为基础的SLR语法分析表构造算法
  -  构造增广文法$G^\prime$的LR(0)项集规范族$\{I_0,I_1,\cdots,I_n\}$
  -  状态i对应项集$I_i$,相关的ACTION/GOTO表条目如下：
     -  $[A \rightarrow \alpha\cdot \textcolor{red}{a}\beta]$在$I_i$中，且$GOTO(I_i,\textcolor{red}{a}) = I_j$,则$ACTION[i,a] = 移入j$
     -  $[A \rightarrow \alpha \cdot]$在$I_i$中，那么对$\textcolor{red}{FOLLOW(A)}$中所有$\textcolor{red}{a}$,$ACTION[i,a] = 按A\rightarrow \alpha 归约$
     -  如果$[S^\prime \rightarrow S\cdot]$在$I_i$中，那么$ACTION[i,\$] = 接受$
     -  如果$GOTO(I_i,A) = I_j$，那么在GOTO表中，$GOTO[i,A] = j$
  - 空白条目设为error
- 如果SLR分析表中没有冲突，那么该文法就是SLR的。

### SLR语法分析器的弱点
- 移入/归约冲突，归约/归约冲突

### 更强大的LR语法分析器
- 规范LR方法
  - 添加项$[A \rightarrow \cdot \alpha]$时，把期望的向前看符号也加入项中
  - 缺点在于状态变多
- 向前看LR方法(LALR方法)
  - 在规范LR方法上压缩了状态，使得分析表和SLR分析表一样大但是分析能力更强。
