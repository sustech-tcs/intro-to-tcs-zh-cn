```admonish warning 
**本章施工中**
```

<!-- toc -->

# 多项式时间归约 {#reductionchap }

## 学习目标 { .objectives }
* 介绍**多项式时间归约**(polynomial time reduction)的概念, 作为将问题的复杂性相互关联的一种方式
* 看到几个此类归约的例子
* 以3SAT作为归约的基本起点

回想我们在第12章遇到的一些问题: 

1. **3SAT**问题: 判断给定的3合取范式(3CNF formula)是否存在可满足赋值. 
2. 在图中寻找**最长路径**. 
3. 在图中寻找**最大割**. 
4. 求解$n$个变量$x_0,\ldots,x_{n-1} \in \R$上的**二次方程组**. 

所有这些问题都具有以下特性: 

* 这些都是重要问题, 人们投入了大量精力试图为它们寻找更好的算法. 
* 这些都是**搜索**问题, 即我们在某个易于定义的层面上(例如长路径、可满足赋值等)搜索一个“好”的解. 
* 每个问题都存在一个平凡的指数时间算法, 即枚举所有可能的解. 
* 目前, 对于所有这些问题, 在最坏情况下, 已知的最好算法并不比平凡算法快多少. 

在本章及[第15章](./chapter_15.md)中, 我们将看到, 尽管这些问题表面差异很大, 但我们可以关联它们以及许多其他问题的计算复杂性. 事实上, 上述问题在**计算上是等价**(computationally equivalent)的, 这意味着解决其中一个问题立刻意味着能解决其他问题. 这种现象被称为$\mathbf{NP}$**完全性**($\mathbf{NP}$ completeness), 是理论计算机科学中一个令人惊讶的发现, 并且我们将看到它具有深远的影响. 

```admonish info title = "简要概述"
本章介绍**多项式时间归约**的概念, 这是计算复杂性理论, 特别是本书的核心概念. 多项式时间归约是一种将解决一个问题的任务**归约**(reduce)到另一个问题的方法. 在复杂性理论中, 我们使用归约来论证: 如果第一个问题难以高效解决, 那么第二个问题也必定难以高效解决. 本章中我们将看到几个归约的例子, 而归约将构成我们在第15章要发展的**$\mathbf{NP}$完全性**理论的基础. 
```

本章描述的所有归约代码可在以下[Jupyter笔记本](https://colab.research.google.com/github/boazbk/tcscode/blob/master/Chap_13_reductions.ipynb)中获取. 

```admonish pic id="reductionsoverviewfig"
![reductionsoverviewfig](./images/chapter14/reductionsoverview.png) 

{{pic}}{fig:reductionsoverview} 在本章中, 我们证明如果$3\text{SAT}$问题不能在多项式时间内解决, 那么$\text{QUADEQ}$、$\text{LONGESTPATH}$、$\text{ISET}$和$\text{MAXCUT}$问题也都不能在多项式时间内解决. 我们通过使用**归约范式**(reduction paradigm)来做到这一点, 例如表明“如果猪能吹口哨”(即如果我们有一个$\text{QUADEQ}$的高效算法), 那么“马就能飞”(即我们将有一个$\text{3SAT}$的高效算法). 
```

在本章中, 我们将看到, 对于在图中寻找最长路径、求解二次方程组以及寻找最大割的每一个问题, 如果存在解决该问题的多项式时间算法, 那么也就存在解决3SAT问题的多项式时间算法. 换句话说, 我们将把解决3SAT的任务**归约**到上述每一项任务. 另一种理解这些结果的方式是, 如果3SAT问题确实**不存在**多项式时间算法, 那么这些其他问题也**不存在**多项式时间算法. 在[第15章](./chapter_15.md)中, 我们将看到证据(尽管不是证明!)表明上述所有问题都不存在多项式时间算法, 因此都是**本质上难解**(inherently intractable)的. 

## 问题的形式化定义 { #formaldefdecisionexamplessec }

出于技术便利性而非实质性的考虑, 我们主要关注**判定问题**(decision problems, 即答案为“是/否”的问题), 或者说**布尔**(Boolean)函数(即输出为一比特的函数). 我们将上述问题建模为从$\{0,1\}^*$到$\{0,1\}$的函数, 方式如下: 

**3SAT: 3SAT问题**可以表述为函数$3\text{SAT}:\{0,1\}^* \rightarrow \{0,1\}$, 其输入为一个3CNF公式$\varphi$(即形如$C_0 \wedge \cdots \wedge C_{m-1}$的公式, 其中每个$C_i$是三个变量或其否定的析取), 并满足: 如果存在对$\varphi$变量的某个赋值使其求值为**真**, 则将$\varphi$映射为$1$; 否则映射为$0$. 例如: 

$$3\text{SAT}\left("(x_0 \vee \overline{x}_1 \vee x_2)  \wedge   (x_1 \vee x_2 \vee \overline{x_3}) \wedge (\overline{x}_0 \vee \overline{x}_2 \vee x_3)" \right)  = 1$$

因为赋值$x = 1101$满足该输入公式. 在上面的描述中, 我们假定公式以某种字符串形式表示, 并约定如果输入不是合法的表示, 则函数输出$0$; 对于下文所有其他函数, 我们采用相同的约定. 

**二次方程组: 二次方程组问题**对应于函数$\text{QUADEQ}:\{0,1\}^* \rightarrow \{0,1\}$, 它将一组二次方程$E$映射为$1$, 如果存在一个赋值$x$满足所有方程; 否则映射为$0$. 

**最长路径: 最长路径问题**对应于函数$\text{LONGPATH}:\{0,1\}^* \rightarrow \{0,1\}$, 它将一个图$G$和一个数$k$映射为$1$, 如果$G$中存在一条长度至少为$k$的简单路径; 否则将$(G,k)$映射为$0$. 最长路径问题是著名的[Hamiltonian路径问题](https://en.wikipedia.org/wiki/Hamiltonian_path_problem)的推广, 后者判定一个给定的$n$顶点图中是否存在一条长度为$n$的路径. 

**最大割: 最大割问题**对应于函数$\text{MAXCUT}:\{0,1\}^* \rightarrow \{0,1\}$, 它将一个图$G$和一个数$k$映射为$1$, 如果$G$中存在一个割能切断至少$k$条边; 否则将$(G,k)$映射为$0$. 

以上所有问题都属于$\mathbf{EXP}$, 但是否属于$\mathbf{P}$尚属未知. 不过, 我们将在本章中看到, 如果$\text{QUADEQ}$、$\text{LONGPATH}$或$\text{MAXCUT}$中的任何一个属于$\mathbf{P}$, 那么$3\text{SAT}$也属于$\mathbf{P}$. 

## 多项式时间归约 {#polytimeredsec }

假设$F,G:\{0,1\}^* \rightarrow \{0,1\}$是两个布尔函数. 从$F$到$G$的**多项式时间归约**(有时简称为 **"归约"**)是一种表明$F$“不难于”$G$的方式, 其含义是: 如果$G$存在多项式时间算法, 则$F$也存在多项式时间算法. 

```admonish quote title=""
{{defc}}{def:reduction-def}[多项式时间归约]

设$F,G:\{0,1\}^* \rightarrow \{0,1\}$. 我们称$F$**可归约到**$G$, 记作$F \leq_p G$, 如果存在一个多项式时间可计算的函数$R:\{0,1\}^* \rightarrow \{0,1\}^*$, 使得对于每个$x\in \{0,1\}^*$, 
$$
F(x) = G(R(x)) \;. {{numeq}}{eq:reduction}
$$
如果$F \leq_p G$且$G \leq_p F$, 我们称$F$和$G$具有**等价的计算复杂度**(equivalent complexity). 
```

```admonish pic id="reductionsfig"
![reductionsfig](./images/chapter14/reductiondescription.png) 

{{pic}}{fig:reductions} 如果$F \leq_p G$, 那么我们可以将计算$G$的多项式时间算法$B$转换为计算$F$的多项式时间算法$A$. 为了计算$F(x)$, 我们可以运行由$F \leq_p G$所保证的归约$R$得到$y=R(x)$, 然后运行我们的算法$B$来计算$G(y)$. 
```

下面的练习证明了我们的直觉: $F \leq_p G$意味着“$F$不难于$G$”. 

```admonish question
{{exec}}{exe:reductionsandP}[归约与$\mathbf{P}$]

证明如果$F \leq_p G$且$G \in \mathbf{P}$, 那么$F\in \mathbf{P}$. 
```

```admonish pause title="暂停一下"
像往常一样, 独立解决这个练习是确保你理解{{ref:def:reduction-def}}的好方法. 
```

```admonish solution collapsible=true title="对{{ref:exe:reductionsandP}}的解答"
假设存在算法$B$在$p(n)$时间内计算$G$, 其中$n$是其输入大小. 那么, {{eqref:eq:reduction}}直接给出了一个计算$F$的算法$A$(见{{ref:fig:reductions}}). 的确, 在输入$x\in \{0,1\}^*$时, 算法$A$将运行多项式时间归约$R$得到$y=R(x)$, 然后返回$B(y)$. 根据{{eqref:eq:reduction}}, $G(R(x)) = F(x)$, 因此算法$A$确实能计算$F$. 

现在证明$A$在多项式时间内运行. 根据假设, $R$可以在$q(n)$时间内计算, 其中$q$是某个多项式. 特别地, 这意味着$|y| \leq q(|x|)$(因为仅仅写出$y$就需要$|y|$步). 计算$B(y)$最多需要$p(|y|) \leq p(q(|x|))$步. 因此, $A$在长度为$n$的输入上的总运行时间最多是计算$y$的时间(以$q(n)$为界)加上计算$B(y)$的时间(以$p(q(n))$为界). 由于两个多项式的复合仍然是多项式, 所以$A$在多项式时间内运行. 
```

```admonish bigidea
{{idec}}{ide:reduction}

**归约**$F \leq_p G$表明$F$“不难于$G$”, 或者说$G$“不易于$F$”. 
```

### 吹口哨的猪与飞翔的马

从$F$到$G$的归约可以用于两个目的: 

* 如果我们已经知道$G$的一个算法, 并且$F \leq_p G$, 那么我们可以利用这个归约得到$F$的一个算法. 这是算法设计中广泛使用的工具. 例如, 在[12.1.4节]()中, 我们看到**最小割最大流定理**(Min-Cut Max-Flow theorem)如何将计算图中最小割的任务归约到计算其中的最大流. 
* 如果我们已经证明(或有证据表明)$F$**不存在**多项式时间算法, 并且$F \leq_p G$, 那么该归约的存在允许我们得出结论: $G$也不存在多项式时间算法. 这是我们在[9.4节](./chapter_9.md#reductionsuncompsec)中看到的“如果猪能吹口哨, 那么马就能飞”的解释. 我们证明, 如果存在一个假设的$G$的高效算法(一只“吹口哨的猪”), 那么由于$F \leq_p G$, 就会存在一个$F$的高效算法(一匹“飞翔的马”). 在本书中, 我们经常将归约用于这第二个目的, 尽管两者之间的界限有时是模糊的(见[14.10节](#reductionsbibnotes)). 

{{ref:def:reduction-def}}中的概念与我们在**不可计算性**背景下(例如[9.4节](./chapter_9.md#reductionsuncompsec))看到的归约之间最关键的差异在于, 为了关联问题的时间复杂度, 我们需要归约是**多项式时间可计算**的, 而不仅仅是可计算的. {{ref:def:reduction-def}}也将归约限制为具有非常特定的形式. 也就是说, 为了证明$F \leq_p G$, 我们只允许通过输出$G(R(x))$来计算$F(x)$的算法, 而不是允许一个使用计算$G$的“神奇黑盒”的通用$F$算法. 这种限制形式对我们来说是方便的, 但人们也定义并使用了更一般的归约(见[14.10节](#reductionsbibnotes)). 

在本章中, 我们使用归约来关联上述问题的计算复杂度: 3SAT、二次方程组、最大割和最长路径, 以及其他一些问题. 我们将把3SAT归约到后面的问题, 证明高效解决其中任何一个问题都将导致3SAT的高效算法. 在[第15章](./chapter_15.md)中, 我们将展示相反的方向: 一举将这些问题中的每一个归约到3SAT. 

**归约的传递性:** 既然我们将$F \leq_p G$理解为(就多项式时间计算而言)$F$“在难度上小于等于”$G$, 那么如果$F \leq_p G$且$G \leq_p H$, 我们期望应有$F \leq_p H$. 事实确实如此: 

```admonish question
{{exec}}{exe:transitiveex}[多项式时间归约的传递性]

对于每个$F,G,H :\{0,1\}^* \rightarrow \{0,1\}$, 如果$F \leq_p G$且$G \leq_p H$, 那么$F \leq_p H$. 
```

```admonish solution collapsible=true title="对{{ref:exe:transitiveex}}的解答"
如果$F \leq_p G$且$G \leq_p H$, 那么存在多项式时间可计算的函数$R_1$和$R_2$, 它们将$\{0,1\}^*$映射到$\{0,1\}^*$, 使得对于每个$x\in \{0,1\}^*$, $F(x) = G(R_1(x))$, 并且对于每个$y\in \{0,1\}^*$, $G(y) = H(R_2(y))$. 结合这两个等式, 我们看到对于每个$x\in \{0,1\}^*$, $F(x) = H(R_2(R_1(x)))$. 因此, 为了证明$F \leq_p H$, 只需证明映射$x \mapsto R_2(R_1(x))$是多项式时间可计算的. 但是, 如果存在某些常数$c,d$使得$R_1(x)$可在$|x|^c$时间内计算, 且$R_2(y)$可在$|y|^d$时间内计算, 那么$R_2(R_1(x))$可在$(|x|^c)^d = |x|^{cd}$时间内计算, 这是多项式时间. 
```

## 将3SAT归约为零一方程与二次方程

我们现在展示归约的第一个例子. **零一线性方程组问题**(Zero-One Linear Equations problem)对应函数$01\text{EQ}:\{0,1\}^* \rightarrow \{0,1\}$, 其输入是变量$x_0,\ldots,x_{n-1}$的一个线性方程组集合$E$, 输出为$1$当且仅当存在一个对变量赋值$0/1$值的$x\in \{0,1\}^n$满足所有方程. 例如, 如果输入$E$是编码以下方程组的字符串

$$
\begin{aligned}
x_0 + x_1 + x_{2} &= 2 \\
x_0 + x_2     &= 1 \\
x_1 + x_{2} &= 2 
\end{aligned}
$$

那么$01\text{EQ}(E)=1$, 因为赋值$x = 011$满足所有三个方程. 我们特别将注意力限制在变量$x_0,\ldots, x_{n-1}$上的线性方程组, 其中每个方程都具有形式$\sum_{i \in S} x_i = b$, 其中$S \subseteq [n]$且$b\in \N$. {{footnote: 如果你熟悉矩阵表示法, 你可能会注意到这样的方程可以写成$Ax = \mathbf{b}$的形式, 其中$A$是一个元素为$0/1$的$m\times n$矩阵, 且$\mathbf{b} \in \N^m$. }}

如果我们问是否存在$x \in \R^n$(**实数**)的解满足$E$, 那么这可以使用著名的**Gaussian消元法**在多项式时间内解决. 然而, 目前没有已知的高效算法来解决$01\text{EQ}$. 事实上, 如以下定理所示, 这样的算法将意味着存在解决$3\text{SAT}$的算法: 

```admonish quote title=""
{{thmc}}{thm:tsattozoeqthm}[$01\text{EQ}$的困难性]
$3\text{SAT} \leq_p 01\text{EQ}$
```

```admonish proof collapsible=true title="{{ref:thm:tsattozoeqthm}}的证明思路"
约束$x_2 \vee \overline{x}_5 \vee x_7$可以写成$x_2 + (1-x_5) + x_7 \geq 1$. 这是一个线性**不等式**, 但由于左侧的和最多为三, 我们也可以通过添加两个新变量$y,z$将其转化为**等式**, 写作$x_2 + (1-x_5) + x_7 + y  + z =3$. (我们将为每个约束使用新的变量$y,z$.)最后, 对于每个变量$x_i$, 我们可以通过添加方程$x_i + x'_i = 1$来添加一个对应其否定的变量$x'_i$, 从而将原始约束$x_2 \vee \overline{x}_5 \vee x_7$映射为$x_2 + x'_5 + x_7 +y + z = 3$. 从这个归约中得到的主要**技术要点**是添加**辅助变量**的思想, 用以替换像$x_1+x_2 +x_3 \geq 1$这样不完全符合我们所需形式的方程, 代之以等价的(对于$0/1$值变量)方程$x_1+x_2+x_3+u+v=3$, 后者符合我们想要的形式. 
```

```admonish pic id="threesat2zoeqreductionfig"
![threesat2zoeqreductionfig](./images/chapter14/3sat2zoeqreduction.png) 

{{pic}}{fig:threesat2zoeqreduction} 左: 实现$3\text{SAT}$到$01\text{EQ}$归约的Python代码. 右: 归约的示例输出. 代码在我们的[代码库](https://github.com/boazbk/tcscode)中. 
```

```admonish proof collapsible=true title="{{ref:thm:tsattozoeqthm}}的证明"
为了证明这个定理, 我们需要: 

1. 描述一个算法$R$, 用于将$3\text{SAT}$的输入$\varphi$映射为$01\text{EQ}$的输入$E$. 
2. 证明该算法在多项式时间内运行. 
3. 证明对于每个3CNF公式$\varphi$, 都有$01\text{EQ}(R(\varphi)) =3SAT(\varphi)$. 

我们现在就来做这些事. 由于这是我们的第一个归约, 我们将详细阐述这个证明. 不过, 它直接遵循上述证明思路. 

{{algc}}{alg:zerooneeqreduction}[$3\text{SAT}$到$01\text{EQ}$的归约]
$$
\begin{align}
&\textbf{输入: }\text{包含}n\text{个变量}x_0,\ldots,x_{n-1}\text{和}m\text{个子句的3CNF公式}\varphi\\
&\textbf{输出: }\text{0/1线性方程组}E\text{, 满足}3\text{SAT}(\varphi)=1\text{当且仅当}01EQ(E)=1\\
&\\
&\text{令}E\text{的变量为}x_0,\ldots,x_{n-1},\; x'_0,\ldots,x'_{n-1},\; y_0,\ldots,y_{m-1},\; z_0,\ldots,z_{m-1}\\
&\for{i \in [n]}\\
&\quad \text{向}E\text{添加方程}x_i + x'_i = 1\\
&\endfor\\
&\for{j\in [m]}\\
&\quad \text{令第}j\text{个子句为}w_0 \vee w_1 \vee w_2\text{, 其中}w_0,w_1,w_2\text{是文字}\\
&\quad \for{a\in[3]}\\
&\quad \quad \if{w_a\text{是变量}x_i}\\
&\quad \qquad t_a \leftarrow x_i\\
&\quad \quad \endif\\
&\quad \quad \if{w_a\text{是变量}\neg x_i}\\
&\quad \qquad t_a \leftarrow x'_i\\
&\quad \quad \endif\\
&\quad \endfor\\
&\quad \text{向}E\text{添加方程}t_0 + t_1 + t_2 + y_j + z_j = 3\\
&\endfor\\
&\return{E}
\end{align}
$$

归约在{{ref:alg:zerooneeqreduction}}中描述, 另见{{ref:fig:threesat2zoeqreduction}}. 如果输入公式有$n$个变量和$m$个子句, {{ref:alg:zerooneeqreduction}}会创建一组$E$, 包含$n+m$个方程, 涉及$2n+2m$个变量. {{ref:alg:zerooneeqreduction}}先进行$n$步的初始循环(每步花费常数时间), 然后进行另一个$m$步的循环(每步花费常数时间)来创建方程, 因此它在多项式时间内运行. 

令$R$为{{ref:alg:zerooneeqreduction}}计算的函数. 证明的核心是要证明对于每个3CNF公式$\varphi$, 都有$01\text{EQ}(R(\varphi)) = 3\text{SAT}(\varphi)$. 我们将证明分为两部分. 第一部分, 传统上称为**完备性**(completeness), 旨在证明如果$3\text{SAT}(\varphi)=1$, 则$01\text{EQ}(R(\varphi))=1$. 第二部分, 传统上称为**可靠性**(soundness), 旨在证明如果$01\text{EQ}(R(\varphi))=1$, 则$3\text{SAT}(\varphi)=1$. ("完备性"和"可靠性"的名称来源于将$R(\varphi)$的解视为$\varphi$可满足的"证明"的观点, 在这种情况下, 这些条件对应于[第11.1.1节]()中定义的完备性和可靠性. 然而, 如果你觉得这些名称令人困惑, 你可以简单地将完备性理解为"$1$-实例映射到$1$-实例"的性质, 将可靠性理解为"$0$-实例映射到$0$-实例"的性质.)

我们通过展示两部分来完成证明: 

* **完备性**:  假设$3\text{SAT}(\varphi)=1$, 这意味着存在一个赋值$x\in \{0,1\}^n$满足$\varphi$. 如果我们对$E=R(\varphi)$的前$2n$个变量使用赋值$x_0,\ldots,x_{n-1}$和$1-x_0,\ldots, 1-x_{n-1}$, 那么我们将满足所有形式为$x_i + x'_i =1$的方程. 此外, 对于每个$j\in [m]$, 如果$t_0 + t_1 + t_2 + y_j + z_j = 3 (*)$是来自$\varphi$第$j$个子句的方程($t_0,t_1,t_2$是形如$x_i$或$x'_i$的变量, 具体取决于子句中的文字), 那么我们对前$2n$个变量的赋值确保$t_0+t_1+t_2 \geq 1$(因为$x$满足$\varphi$), 因此我们可以为$y_j$和$z_j$赋值, 以确保方程$(*)$被满足. 因此在这种情况下, $E = R(\varphi)$是可满足的, 意味着$01\text{EQ}(R(\varphi))=1$. 
* **可靠性**:  假设$01\text{EQ}(R(\varphi))=1$, 这意味着方程组$E=R(\varphi)$有一个满足条件的赋值$x_0,\ldots,x_{n-1}$,$x'_0,\ldots,x'_{n-1}$,$y_0,\ldots,y_{m-1}$,$z_0,\ldots,z_{m-1}$. 那么, 由于方程包含条件$x_i + x'_i = 1$, 对于每个$i \in [n]$, $x'_i$都是$x_i$的否定, 并且, 对于每个$j\in [m]$, 如果$C$具有形式$w_0 \vee w_1 \vee w_2$且是$\varphi$的第$j$个子句, 那么相应的赋值$x$将确保$w_0 + w_1 + w_2 \geq 1$, 这意味着$C$被满足. 因此在这种情况下$3\text{SAT}(\varphi)=1$. 
```

### 二次方程组

既然我们已经将$3\text{SAT}$归约到$01\text{EQ}$, 那么我们可以利用这一点, 进一步将$3\text{SAT}$归约到**二次方程组**问题(quadratic equations problem). 这对应于函数$\text{QUADEQ}$, 其输入是一个[次数](https://en.wikipedia.org/wiki/Degree_of_a_polynomial)最多为2的$n$元多项式列表$p_0,\ldots,p_{m-1}:\R^n \rightarrow \R$(即它们是**二次的**), 且多项式均具有整数系数. (后一个条件是为了方便, 可以通过缩放实现.)我们定义$\text{QUADEQ}(p_0,\ldots,p_{m-1})$等于$1$当且仅当 存在一个解$x\in \R^n$满足方程组$p_0(x)=0$,$p_1(x)=0$,$\ldots$,$p_{m-1}(x)=0$. 

例如, 以下是关于变量$x_0,x_1,x_2$的一组二次方程: 
$$
\begin{aligned}
x_0^2 - x_0 &= 0 \\
x_1^2 - x_1 &= 0 \\
x_2^2 - x_2 &= 0 \\
1-x_0-x_1+x_0x_1    &= 0
\end{aligned}
$$
你可以验证$x \in \R^3$满足这组方程当且仅当$x \in \{0,1\}^3$且$x_0 \vee x_1 = 1$. 

```admonish quote title=""
{{thmc}}{thm:quadeq-thm}[二次方程组问题的困难性]

$$3\text{SAT} \leq_p \text{QUADEQ}$$
```

```admonish proof collapsible=true title="{{ref:thm:quadeq-thm}}的证明思路"
利用归约的传递性({{ref:exe:transitiveex}}), 只需证明$01\text{EQ} \leq_p \text{QUADEQ}$即可, 而这一点成立是因为我们可以将方程$x_i \in \{0,1\}$表达为二次约束$x_i^2 - x_i = 0$. 此归约的**核心技巧**在于, 我们可以利用**非线性性**来强制连续变量(例如, 取值于$\R$的变量)变为离散的(例如, 取值于$\{0,1\}$). 
```

```admonish proof collapsible=true title="{{ref:thm:quadeq-thm}}的证明"
根据{{ref:thm:tsattozoeqthm}}和{{ref:exe:transitiveex}}, 只需证明$01\text{EQ} \leq_p \text{QUADEQ}$. 设$E$是一个$01\text{EQ}$的实例, 其变量为$x_0,\ldots,x_{n-1}$. 我们将$E$映射到二次方程组$E'$, 该方程组是通过取$E$中的线性方程, 并为其添加$n$个二次方程$x_i^2 - x_i = 0$(对于所有$i\in [n]$)而得到的. (参见算法 14.5)映射$E \mapsto E'$可以在多项式时间内计算. 我们断言$01\text{EQ}(E)=1$当且仅当$\text{QUADEQ}(E')=1$. 实际上, 两个实例之间的唯一区别在于: 

* 在$01\text{EQ}$实例$E$中, 方程是关于变量$x_0,\ldots,x_{n-1} \in \{0,1\}$的. 
* 在$\text{QUADEQ}$实例$E'$中, 方程是关于变量$x_0,\ldots,x_{n-1} \in \R$的, 但我们有额外的约束$x_i^2 - x_i = 0$(对于所有$i\in [n]$). 

因为对于每个$a\in \R$, $a^2 - a = 0$当且仅当$a \in \{0,1\}$, 所以这两组方程是等价的, 因此$01\text{EQ}(E)=\text{QUADEQ}(E')$, 这正是我们需要证明的. 

{{algc}}{alg:zeroonetoquadreductionalg}[$01\text{EQ}$到$\text{QUADEQ}$的归约]
$$
\begin{align}
&\textbf{输入: }\text{包含}n\text{个变量}x_0,\ldots,x_{n-1}\text{的线性方程组}E\\
&\textbf{输出: }\text{包含}m\text{个变量}w_0,\ldots,w_{m-1}\text{的二次方程组}E'\\
&\text{, 满足存在一个0/1赋值}x\in \{0,1\}^n\\
&\text{满足}E\text{的方程当且仅当存在一个赋值}w \in \R^m\text{满足}E'\text{的方程. }\\
&\text{即, }01\text{EQ}(E) = \text{QUADEQ}(E')\\
&\\
&\text{令}m\gets n\\
&\text{将}E'\text{的变量设置为与}E\text{相同的变量}x_0,\ldots, x_{n-1}\\
&\for{\text{每个}e\in E}\\
&\quad \text{将}e\text{添加到}E'\\
&\endfor\\
&\for{i\in [n]}\\
&\quad \text{将方程}x_i^2 - x_i = 0\text{添加到}E'\\
&\endfor\\
&\return{E'}
\end{align}
$$
```

## 子集和问题

作为$3\text{SAT}$到$01\text{EQ}$归约的另一个结果, 我们也可以证明$3\text{SAT}$(通过$01\text{EQ}$)可以归约到**子集和**问题(subset sum problem)(也称为**背包**问题knapsack problem). 在**子集和**问题中, 我们有一个整数列表$x_0,\ldots,x_{n-1} \in \mathbb{Z}$和一个整数$T\in\mathbb{Z}$. 我们需要判断是否存在某个整数子集, 其和恰好等于$T$. 也就是说, 对于$x_0,\ldots,x_{n-1},T \in \mathbb{Z}$, $\text{SSUM}(x_0,\ldots,x_{n-1},T)=1$当且仅当存在$S\subseteq [n]$使得$\sum_{i\in S} x_i = T$. 注意, 子集和问题的输入长度是编码所有数字所需的字符串长度, 大约为$\lceil \log T \rceil + \sum_{i=0}^n \lceil \log x_i \rceil$, 因为使用二进制表示编码一个整数$x$需要$\lceil \log x \rceil$位. 

```admonish quote title=""
{{thmc}}{thm:subsetsum-thm}[子集和问题的困难性]

$$3\text{SAT} \leq_p \text{SSUM}$$
```

```admonish proof collapsible=true title="{{ref:thm:subsetsum-thm}}的证明思路"
我们从$01\text{EQ}$进行归约. 直观想法如下. 考虑一个$01\text{EQ}$实例$E$, 它有$n$个变量$x_0,\ldots,x_{n-1}$和$m$个方程$e_0,\ldots,e_{m-1}$. 回想一下, $E$中的每个方程$e_\ell$都具有$x_i + x_j +  x_k = b$的形式(左边求和的变量可能多于或少于三个). 对于每个变量$x_i$, 我们可以定义一个向量$v^i \in \{0,1\}^m$, 其中$v^i_t=1$表示变量$x_i$出现在方程$e_t$中, 否则$v^i_t=0$. 那么, 方程组存在解当且仅当存在某个集合$S\subseteq [n]$(对应于那些$x_i=1$的$i$)使得$\sum_{i\in S} v^i = \vec{b}$, 其中$\vec{b} \in \mathbb{Z}^m$是方程右边常数项构成的向量(即$\vec{b}_t$是第$t$个方程右边的值$b_t$). 现在, 如果我们能够将向量$v^0,\ldots,v^{n-1}$和$\vec{b}$解释为**数字**, 那么我们就可以将其视为一个子集和实例. 关键见解是, 我们确实可以通过将向量$v$的第$j$个坐标视为第$j$位数字来将向量看作数字. 由于向量属于$\{0,1\}^m$, 自然的选择是使用二进制基数, 但这在相加时会导致"进位"问题. 因此, 我们使用一个更大的基数$B$, 详见以下证明. 
```

```admonish proof collapsible=true title="{{ref:thm:subsetsum-thm}}的证明"
对于给定的$01\text{EQ}$方程组(包含$n$个变量), 我们注意到右边常数项的值永远不会大于$n$(因为最多$n$个在$\{0,1\}$中的变量之和最多为$n$). 更具体地说, 如果实例中存在这样的方程, 那么我们可以确定答案为$0$(在归约的上下文中, 可以将其映射到某个没有解的子集和简单实例, 例如$x_0=x_1=1$且$T=3$). 

我们的归约如{{ref:alg:zeroonetossumnalg}}所述. 对于输入的一个$01\text{EQ}$实例$E = \{ e_t \}_{t=1}^m$(包含$n$个变量$x_0,\ldots,x_{n-1}$), 我们输出一个$\text{SSUM}$实例$y_0,\ldots,y_{n-1},T$, 计算如下: 

* $y_i = \sum_{t=0}^{m-1} B^t v^i_t$, 其中$v^i_t$等于$1$如果变量$x_i$出现在方程$e_t$中, 否则等于$0$. 数$B$被设为$2n$(任何大于$n$的数都可以). 
* $T = \sum_{t=0}^{m-1} B^t b_t$, 其中$b_t$是方程$e_t$右边的整数. 

换句话说, $y_0,\ldots,y_{n-1}$和$T$是满足以下条件的整数: 在$B$进制表示下, $y_i$的第$t$位数字是$1$当且仅当$x_i$出现在$x_t$中, 而$T$的第$t$位数字是$e_t$的右边常数项. 

以下断言将蕴含归约的正确性: 

**断言**:  对于每个$x\in \{0,1\}^n$, 如果$S = \{ i | x_i = 1 \}$, 那么$x$满足$E$的方程当且仅当$\sum_{i\in S} y_i = T$. 

**证明**:  证明的关键在于以下简单的加法性质: 当在$B$进制下相加最多$n$个数时, 如果所有这些数的所有位数字都是$0$或$1$, 并且$B>n$, 那么对于每个$t$, 和数的第$t$位数字就是这些数第$t$位数字的和. 这是因为加法中没有"进位". 在我们的例子中, 数字$y_0,\ldots,y_n$在$B$进制下满足此性质, 且$B>n$, 因此对于每个$S \subseteq [n]$和每个位数字$t$, 和$\sum_{i\in S}y_i$的第$t$位数字就仅仅是这些数第$t$位数字的和, 这对应于所有参与第$t$个方程的$x_i$之和. 当且仅当该方程被满足时, 此和才等于$T$的第$t$位数字. 

该断言表明$01\text{EQ}(E) = \text{SSUM}(y_0,\ldots,y_{n-1},T)$, 这正是我们需要证明的. 

{{algc}}{alg:zeroonetossumnalg}[$01\text{EQ}$到$\text{SSUM}$的归约]
$$
\begin{align}
&\textbf{输入: }\text{包含}n\text{个变量}x_0,\ldots,x_{n-1}\text{的}m\text{个线性方程组成的集合}E = \{ e_t \}_{t\in [m]}\\
&\textbf{输出: }\text{整数}y_0,\ldots,y_{n-1},T \in \mathbb{Z}\text{, 满足存在一个0/1赋值}x\in \{0,1\}^n\\
&\text{满足}E\text{的方程当且仅当存在}S \subseteq [n]\text{使得}\sum_{i\in S}y_i = T\\
&\\
&\for{t\in [m]}\\
&\quad \text{令}A \subseteq [n]\text{和}b\in \mathbb{Z}\text{使得}e_t\text{具有形式}\sum_{i\in A} x_i = b\\
&\quad \text{如果}i\in A\text{, 则令}v_i^t \leftarrow 1\text{; 否则令}v_i^t \leftarrow 0\\
&\quad \text{令}b_t  \leftarrow  b\\
&\endfor\\
&\text{令}B \leftarrow 2n\\
&\for{i\in [n]}\\
&\quad \text{令}y_i \leftarrow \sum_{t=1}^m B^t v_i^t\\
&\endfor\\
&\text{令}T \leftarrow  \sum_{t=1}^T B^t b_t\\
&\return{y_0,\ldots,y_{n-1},T}
\end{align}
$$
```

## 独立集问题

对于图$G=(V,E)$, [独立集](https://en.wikipedia.org/wiki/Independent_set_(graph_theory))(亦称**稳定集**stable set)是指子集$S \subseteq V$, 满足$S$中任意两点间均无边相连(换言之, $E(S,S)=\emptyset$). 每个“单点集”(singleton, 仅包含单个顶点的集合)显然都是独立集, 但寻找更大的独立集可能颇具挑战性. **最大独立集**问题(maximum independent set problem, 下文简称“独立集问题”)旨在寻找图中规模最大的独立集. 该问题天然关联于**调度问题**: 若在相互冲突的两个任务间连边, 则独立集对应于可无冲突同时调度的一组任务. 独立集问题已在多种场景中得到研究, 例如在[蛋白质相互作用图](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3919085/)中发现结构的算法. 

如[第14.1节](#formaldefdecisionexamplessec)所述, 我们将独立集问题视为函数$ISET:\{0,1\}^* \rightarrow \{0,1\}$: 输入图$G$与数值$k$, 当且仅当$G$包含规模至少为$k$的独立集时输出$1$. 现在我们将3SAT归约至独立集问题. 

```admonish quote title=""
{{thmc}}{thm:isetnpc}[独立集问题的困难性]

$3\text{SAT} \leq_p \text{ISET}$
```

```admonish proof collapsible=true title="{{ref:thm:isetnpc}}的证明思路"
核心思想在于: 为3SAT公式寻找满足赋值, 对应于在避免冲突的前提下满足多个局部约束. 可将“$x_{17}=0$”与“$x_{17}=1$”视为两个冲突事件, 而约束$x_{17} \vee \overline{x}_5 \vee x_9$则建立了事件“$x_{17}=0$”“$x_5=1$”“$x_9=0$”间的冲突, 意味着三者不能同时发生. 由此, 可将解决3SAT问题理解为调度无冲突事件的过程, 当然关键难点仍在于细节处理. 此处的**核心技术**是将原公式的每个子句映射为一个**构件**(gadget)——即满足特定性质的小型子图(或更广义的“子实例”). 我们将在多项式时间归约的构造中反复看到这类“构件”的应用. 
```

```admonish pic id="example3sat2isetfig"
![example3sat2isetfig](./images/chapter14/example3sat2iset.png) 

{{pic}}{fig:example3sat2iset} 将$3\text{SAT}$归约至$\text{ISET}$的示例, 原始输入公式为$\varphi = (x_0 \vee \overline{x}_1 \vee x_2) \wedge (\overline{x}_0 \vee x_1 \vee \overline{x}_2) \wedge (x_1 \vee x_2 \vee \overline{x}_3)$. 我们将$\varphi$的每个子句映射为由三个顶点构成的三角形, 每个顶点根据使对应文字为真的$x_i$取值标记为“$x_i = 0$”或“$x_i=1$”. 在所有**冲突的**文字变量对(即分别标记为“$x_i=0$”和“$x_i=1$”的顶点)间添加边. 
```

```admonish quote title=""
{{algc}}{alg:threesattoisetreductionalg}[$3\text{SAT}$到$\text{IS}$的归约]
$$
\begin{align}
&\textbf{输入: }\text{包含}n\text{个变量和}m\text{个子句的3SAT公式}\varphi\\
&\textbf{输出: }\text{图}G=(V,E)\text{与数值}k\text{, 满足}G\text{存在规模为}k\text{的独立集当且仅当}\varphi\text{满足赋值.}\\
&\text{即}3\text{SAT}(\varphi) = \text{ISET}(G,k)\\
&\\
&\text{初始化}V \leftarrow \emptyset, E \leftarrow \emptyset\\
&\for{\text{每个子句}C = y \vee y' \vee y''}\\
&\quad \text{将三个顶点}(C,y),(C,y'),(C,y'')\text{加入}V\\
&\quad \text{将边}\{ (C,y), (C,y') \}, \{(C,y'),(C,y'') \}, \{ (C,y''), (C,y) \}\text{加入}E\\
&\endfor\\
&\for{\text{对}\varphi\text{中每对不同的子句}C,C'}\\
&\quad \for{每个i\in [n]}\\
&\quad \quad \if{C\text{包含文字}x_i\text{且}C'\text{包含文字}\overline{x}_i}\\
&\quad \qquad \text{将边}\{ (C,x_i), (C',\overline{x}_i) \}\text{加入}E\\
&\quad \quad \endif\\
&\quad \endfor\\
&\endfor\\
&\return{(G=(V,E), m)}
\end{align}
$$
```

```admonish proof collapsible=true title="{{ref:thm:isetnpc}}的证明"
给定包含$n$个变量与$m$个子句的3SAT公式$\varphi$, 我们将按如下方式构造包含$3m$个顶点的图$G$(参见{{ref:alg:threesattoisetreductionalg}}, {{ref:fig:example3sat2iset}}为示例, {{ref:fig:threesattois}}为Python代码实现): 

* $\varphi$中的子句$C$具有形式$C = y \vee y' \vee y''$, 其中$y,y',y''$为_文字变量_(变量或其否定). 对每个这样的子句$C$, 我们在$G$中添加三个顶点, 并分别标记为$(C,y)$、$(C,y')$和$(C,y'')$. 同时在这三个顶点两两之间添加边, 使其构成**三角形**. 由于$\varphi$包含$m$个子句, 图$G$将包含$3m$个顶点. 
* 除上述边外, 我们还在所有形式为$(C,y)$和$(C',y')$的顶点对间添加边, 其中$y$与$y'$是**冲突的**文字变量. 即若存在$i$使得$y=x_i$而$y' = \overline{x}_i$(或反之), 则在$(C,y)$与$(C',y')$间添加边. 

基于$\varphi$构造$G$的算法耗时多项式级别: 包含两个循环, 第一个循环需要$O(m)$步, 第二个循环需要$O(m^2 n)$步(见{{ref:alg:threesattoisetreductionalg}}). 因此要证明定理, 只需证$\varphi$可满足当且仅当$G$包含$m$个顶点的独立集. 现证明该等价关系的两个方向: 

**第一部分: 完备性**. “完备性”方向需证明: 若$\varphi$存在满足赋值$x^*$, 则$G$存在包含$m$个顶点的独立集$S^*$. 现证明如下: 

假设$\varphi$存在满足赋值$x^* \in \{0,1\}^n$. 则对$\varphi$的每个子句$C = y \vee y' \vee y''$, 文字变量$y,y',y''$中至少有一个在赋值$x^*$下计算为**真**(否则无法满足$\varphi$). 我们构造包含$m$个顶点的集合$S$: 对每个子句$C$, 选取一个形式为$(C,y)$且$y$在$x^*$下计算为真的顶点加入$S$(若同一子句存在多个满足条件的顶点, 则任选其一). 

我们断言$S$是独立集. 假设存在$S$中的顶点对$(C,y)$和$(C',y')$之间有边相连. 由于我们从每个子句对应的三角形中仅选取了一个顶点, 必有$C \neq C'$. 此时$(C,y)$和$(C',y')$之间存在边的唯一可能是$y$与$y'$为冲突文字变量(即存在$i$使$y=x_i$且$y'=\overline{x}_i$). 但这样它们在赋值$x^*$下不可能同时计算为**真**, 与集合$S$的构造方式矛盾. 完备性得证. 

**第二部分: 可靠性**. “可靠性”方向需证明: 若$G$存在包含$m$个顶点的独立集$S^*$, 则$\varphi$存在满足赋值$x^* \in \{0,1\}^n$. 现证明如下: 

假设$G$存在包含$m$个顶点的独立集$S^*$. 我们按如下规则为$\varphi$的变量定义赋值$x^* \in \{0,1\}^n$: 
* 若$S^*$包含形式为$(C,x_i)$的顶点, 则令$x^*_i=1$
* 若$S^*$包含形式为$(C,\overline{x_i})$的顶点, 则令$x^*_i=0$
* 若$S^*$不包含上述两种形式的顶点, 则$x^*_i$可任意取值(为明确起见取$x^*_i=0$)

首先注意到$x^*$是良定义的: 上述规则不会相互冲突, 不会要求$x^*_i$同时取$0$和$1$. 这是因为$S^*$是**独立集**, 若其包含$(C,x_i)$形式的顶点, 则不可能包含$(C',\overline{x_i})$形式的顶点. 

现断言$x^*$是$\varphi$的满足赋值. 由于$S^*$是独立集, 在每个对应于$\varphi$子句的三角形$(C,y),(C,y'),(C,y'')$中, $S^*$至多包含一个顶点. 又因$|S^*|=m$, 故在每个此类三角形中恰包含一个顶点. 对$\varphi$的每个子句$C$, 若$(C,y)$是$S^*$在对应三角形中的顶点, 则根据$x^*$的定义, 文字变量$y$必计算为**真**, 这意味着$x^*$满足该子句. 因此$x^*$满足$\varphi$的所有子句, 即满足赋值的定义. 

定理14.8证毕. 
```

```admonish pic id="threesattoisfig"
![threesattoisfig](./images/chapter14/3sat2ISreduction.png) 

{{pic}}{fig:threesattois} 3SAT到独立集的归约示意图. 右侧为实施该归约的**Python**代码, 左侧为归约的样例输出. 黑色边表示“三角形边”, 红色边表示“冲突边”. 注意满足赋值$x^* = 0110$对应于独立集$(0,\neg x_3)$、$(1, \neg x_0)$、$(2,x_2)$. 
```

## 归约的若干练习与结构剖析

归约可能令人困惑, 而通过练习是增进对其熟悉度的绝佳方式. 这里提供这样一个示例. 一如既往, 我建议你在查看解答之前先自行尝试. 

```admonish question
{{exec}}{exe:vertexcoverex}[顶点覆盖问题]

图$G=(V,E)$中的一个**顶点覆盖**(vertex cover)是顶点的一个子集$S \subseteq V$, 使得每条边都至少与$S$中的一个顶点相连(见{{ref:fig:smallvertexcover}}). **顶点覆盖问题**的任务是, 给定图$G$和一个数$k$, 判断图中是否存在一个最多包含$k$个顶点的顶点覆盖. 形式化地说, 这是函数$\text{VC}:\{0,1\}^* \rightarrow \{0,1\}$, 使得对于每个$G=(V,E)$和$k\in \N$, $\text{VC}(G,k)=1$当且仅当存在一个顶点覆盖$S \subseteq V$满足$|S| \leq k$. 

证明$3\text{SAT} \leq_p \text{VC}$. 
```

```admonish pic id="smallvertexcoverfig"
![smallvertexcoverfig](./images/chapter14/vertex_cover.png) 

{{pic}}{fig:smallvertexcover} 图中的**顶点覆盖**是一个与所有边相连的顶点子集. 在这个包含$7$个顶点的图中, $3$个填充顶点构成一个顶点覆盖. 
```

```admonish solution collapsible=true title="对{{ref:exe:vertexcoverex}}的解答"
关键观察是: 如果$S \subseteq V$是一个触及所有顶点的顶点覆盖, 那么不存在边$e$使得$e$的两个端点都在集合$\overline{S} = V \setminus S$中, 反之亦然. 换言之, $S$是顶点覆盖当且仅当$\overline{S}$是独立集. 由于$\overline{S}$的大小是$|V|-|S|$, 我们看到多项式时间映射$R(G,k)=(G,n-k)$(其中$n$是$G$的顶点数)满足$\text{VC}(R(G,k))= \text{ISET}(G,k)$, 这意味着这是一个从独立集到顶点覆盖的归约. 
```

```admonish question
{{exec}}{exe:iscliqueex}[团与独立集的等价性]

最大团问题对应于函数$\text{CLIQUE}:\{0,1\}^* \rightarrow \{0,1\}$, 使得对于图$G$和数$k$, $\text{CLIQUE}(G,k)=1$当且仅当存在一个包含$k$个顶点的子集$S$, 使得对于**任意**不同的$u,v \in S$, 边$u,v$都在$G$中. 这样的集合称为一个**团**(clique). 

证明$\text{CLIQUE} \leq_p \text{ISET}$和$\text{ISET} \leq_p \text{CLIQUE}$. 
```

```admonish solution collapsible=true title="对{{ref:exe:iscliqueex}}的解答"
如果$G=(V,E)$是一个图, 我们用$\overline{G}$表示它的**补图**(complement), 该图具有相同的顶点集$V$, 并且对于任意不同的$u,v \in V$, 边$\{u,v\}$出现在$\overline{G}$中当且仅当该边**不**出现在$G$中. 

这意味着对于任意集合$S$, $S$是$G$中的独立集当且仅当$S$是$\overline{G}$中的一个**团**. 因此对于每个$k$, $\text{ISET}(G,k)=\text{CLIQUE}(\overline{G},k)$. 由于映射$G \mapsto \overline{G}$可以被高效计算, 这就产生了一个归约$\text{ISET} \leq_p \text{CLIQUE}$. 此外, 由于$\overline{\overline{G}}=G$, 这也就产生了另一个方向的归约. 
```

### 支配集

在上述两个例子中, 归约几乎是"平凡的": 从独立集到顶点覆盖的归约仅仅将数$k$改为$n-k$, 而从独立集到团的归约则将边翻转为非边, 反之亦然. 下面的练习需要一个更有趣一些的归约. 

```admonish question
{{exec}}{exe:dominatingsetex}[支配集]

图$G=(V,E)$中的一个**支配集**(dominating set)是顶点的一个子集$S \subseteq V$, 使得每个$u \in V \setminus S$都是$G$中某个$s \in S$的邻居(见{{ref:fig:dominatingvertexcover}}). **支配集问题**的任务是, 给定图$G=(V,E)$和数$k$, 判断是否存在一个支配集$S \subseteq V$满足$|S| \leq k$. 形式化地说, 这是函数$\text{DS}:\{0,1\}^* \rightarrow \{0,1\}$, 使得$\text{DS}(G,k)=1$当且仅当$G$中存在一个最多包含$k$个顶点的支配集. 

证明$\text{ISET} \leq_p \text{DS}$. 
```

```admonish pic id="dominatingvertexcoverfig"
![dominatingvertexcoverfig](./images/chapter14/dominatingvc.png) 

{{pic}}{fig:dominatingvertexcover} 支配集是顶点的子集$S$, 使得图中每个顶点要么在$S$中, 要么是$S$中某个顶点的邻居. 上图中的两个子图是同一个图. 左侧的红色顶点是一个顶点覆盖, 但不是支配集. 右侧的蓝色顶点是一个支配集, 但不是顶点覆盖. 
```

```admonish solution collapsible=true title="对{{ref:exe:dominatingsetex}}的解答"
既然我们知道$\text{ISET} \leq_p \text{VC}$, 利用传递性, 只需证明$\text{VC} \leq_p \text{DS}$. 如{{ref:fig:dominatingvertexcover}}所示, 支配集与顶点覆盖不是同一个概念. 然而, 我们仍然可以关联这两个问题. 思路是将图$G$映射为图$H$, 使得$G$中的顶点覆盖能够转化为$H$中的支配集, 反之亦然. 我们的做法是: 在$H$中包含$G$的所有顶点和边, 但对于$G$中的每条边$\{u ,v \}$, 我们还向$H$添加一个新顶点$w_{u,v}$并将其同时连接到$u$和$v$. 设$\ell$为$G$中孤立顶点的数量. 证明背后的思路是: 我们可以通过向$S$添加所有孤立顶点, 将$G$中大小为$k$的顶点覆盖$S$转化为$H$中大小为$k+\ell$的支配集; 而且, 我们可以将$H$中每个大小为$k+\ell$的支配集转化为$G$中的顶点覆盖. 现在我们给出细节. 

**算法描述**: 给定顶点覆盖问题的一个实例$(G,k)$, 我们将$G$映射为支配集问题的一个实例$(H,k')$, 具体如下(Python实现见{{ref:fig:vctodsreduction}}): 

{{algc}}{alg:independentsettodsredalg}[$\text{VC}$到$\text{DS}$的归约]
$$
\begin{align}
&\textbf{输入: }\text{图}G=(V,E)\text{和数}k\\
&\textbf{输出: }\text{图}H=(V',E')\text{和数}k'\text{, 满足}G\text{存在规模为}\\
&k\text{的顶点覆盖当且仅当}H\text{存在规模为}k'\text{的支配集. }\\
&\text{即}\text{DS}(H,k') = \text{VC}(G,k)\\
&\\
&\text{初始化}V' \leftarrow V, E' \leftarrow E\\
&\for{\text{每条边}\{u,v\} \in E}\\
&\quad \text{将顶点}w_{u,v}\text{加入}V'\\
&\quad \text{将边}\{ u, w_{u,v} \},\{ v, w_{u,v} \}\text{加入}E'\\
&\endfor\\
&\text{令}\ell \leftarrow \text{G中孤立顶点的数量}\\
&\return{(H=(V',E'), k+\ell)}
\end{align}
$$

{{ref:alg:independentsettodsredalg}}在多项式时间内运行, 因为循环需要$O(m)$步, 其中$m$是边的数量, 每一步可以在常数或至多线性时间内实现(取决于图$H$的表示方式). 计算$n$顶点图$G$中孤立顶点的数量, 如果$G$用邻接矩阵表示, 则可以在$O(n^2)$时间内完成; 如果用邻接表表示, 则可以在$O(n)$时间内完成. 无论如何, 该算法在多项式时间内运行. 

为了完成证明, 我们需要证明对于每个$G,k$, 如果$H,k'$是{{ref:alg:independentsettodsredalg}}在输入$(G,k)$上的输出, 那么$\text{DS}(H,k') = \text{VC}(G,k)$. 我们将证明分为两部分. **完备性**部分是指如果$\text{VC}(G,k)=1$那么$\text{DS}(H,k')=1$. **可靠性**部分是指如果$\text{DS}(H,k')=1$那么$VC(G,k)=1$. 

**完备性**: 假设$\text{VC}(G,k)=1$. 那么存在一个最多包含$k$个顶点的顶点覆盖$S \subseteq V$. 设$I$是$G$中孤立顶点的集合, $\ell$是它们的数量. 那么$|S \cup I| \leq k +\ell$. 我们声称$S \cup I$是$H$中的一个支配集. 确实, 对于$H$的每个顶点$v$, 有三种情况: 

* **情况1**: $v$是$G$中的孤立顶点. 这种情况下$v$在$S \cup I$中. 
* **情况2**: $v$是$G$中的非孤立顶点, 因此存在$G$的某条边$\{ u,v \}$. 这种情况下, 由于$S$是顶点覆盖, $u,v$中必须有一个在$S$中, 因此$v$或$v$的一个邻居必须在$S \subseteq S \cup I$中. 
* **情况3**: $v$是$w_{u,u'}$这种形式的顶点, 其中$u,u'$是$G$中的两个邻居. 但由于$S$是顶点覆盖, $u,u'$中必须有一个在$S$中, 因此$S$包含$v$的一个邻居. 

我们得出结论: $S \cup I$是$H'$中一个大小不超过$k'=k +\ell$的支配集, 因此在假设$\text{VC}(G,k)=1$的条件下, $\text{DS}(H',k')=1$. 

**可靠性**: 假设$\text{DS}(H,k')=1$. 那么$H$中存在一个大小不超过$k' = k +\ell$的支配集$D$. 对于图$G$中的每条边$\{ u,v \}$, 如果$D$包含顶点$w_{u,v}$, 那么我们将这个顶点移除并用$u$代替它. $w_{u,v}$仅有的两个邻居是$u$和$v$, 并且由于$u$既是$w_{u,v}$的邻居也是$v$的邻居, 用$u$替换$w_{u,v}$保持了它是一个支配集的性质. 此外, 这个改变不会增加$D$的大小. 因此, 经过这个修改后, 我们可以假设$D$是一个最多包含$k+\ell$个顶点且不包含任何$w_{u,v}$形式顶点的支配集. 

设$I$是$G$中的孤立顶点集合. 这些顶点在$H$中也是孤立的, 因此必须包含在$D$中(孤立顶点必须包含在任何支配集中, 因为它没有邻居). 我们令$S = D \setminus I$. 那么$|S| \leq k$. 我们声称$S$是$G$中的一个顶点覆盖. 确实, 对于$G$中的每条边$\{u,v\}$, 根据支配集性质, 要么顶点$w_{u,v}$, 要么它的某个邻居必须在$S$中. 但由于我们确保了$S$不包含任何$w_{u,v}$形式的顶点, 那么$u$或$v$必须有一个在$S$中. 这表明$S$是$G$中一个大小不超过$k$的顶点覆盖, 从而证明$\text{VC}(G,k)=1$. 

{{ref:alg:independentsettodsredalg}}及我们目前所见其他归约的一个推论是: 如果$\text{DS} \in \mathbf{P}$(即支配集存在多项式时间算法), 那么$3\text{SAT} \in \mathbf{P}$(即$3\text{SAT}$存在多项式时间算法). 根据逆否命题, 如果$3\text{SAT}$**不**存在多项式时间算法, 那么支配集也不存在. 
```

```admonish pic id="vctodsreductionfig"
![vctodsreductionfig](./images/chapter14/vctodsreduction.png) 

{{pic}}{fig:vctodsreduction} 从顶点覆盖到支配集的归约的Python实现, 以及一个输入图和相应输出图的示例. 这个归约允许将假想的支配集多项式时间算法("吹口哨的猪")转化为假想的顶点覆盖多项式时间算法("飞行的马"). 
```

### 归约的结构剖析

```admonish pic id="reductionanatomyfig"
![reductionanatomyfig](./images/chapter14/reductionanatomy.png) 

{{pic}}{fig:reductionanatomy} 归约的四个组成部分, 以顶点覆盖问题归约到支配集问题的具体例子进行说明. 从问题$F$到问题$G$的一个归约, 是一个将$F$的输入$x$映射为$G$的输入$R(x)$的算法. 为了证明该归约是正确的, 我们需要证明以下性质: **高效性**: 算法$R$在多项式时间内运行; **完全性**: 如果$F(x)=1$, 则$G(R(x))=1$; **可靠性**: 如果$F(R(x))=1$, 则$G(x)=1$. 
```

{{ref:exe:dominatingsetex}}中的归约很好地展示了归约的组成部分. 一个归约包含四个部分: 

*   **算法描述**:  这部分描述算法**如何**将输入映射为输出. 例如, 在{{ref:exe:dominatingsetex}}中, 就是描述我们如何将**顶点覆盖**问题的一个实例$(G, k)$映射为**支配集**问题的一个实例$(H, k')$. 
*   **算法分析**:  仅仅描述算法**如何**工作是不够的, 我们还需要解释**为什么**它能工作. 具体来说, 我们需要提供一项**分析**, 解释为什么该归约既是**高效的**(即在多项式时间内运行)又是**正确的**(满足对每个$x$都有$G(R(x))=F(x)$). 具体而言, 对一个归约$R$的分析包含以下部分: 
    *   **高效性**:  我们需要证明$R$在多项式时间内运行. 在遇到的大多数归约中, 这部分是直截了当的, 因为我们通常使用的归约只涉及常数数量的嵌套循环, 每个循环包含常数数量的操作. 例如, {{ref:exe:dominatingsetex}}中的归约仅仅是枚举输入图的边和顶点. 
    *   **完全性**:  在证明$F \leq_p G$的归约$R$中, **完全性**条件是指: 对于每个$x \in \{0,1\}^*$, 如果$F(x) = 1$, 那么$G(R(x))=1$. 我们通常通过将证明$F(x)=1$的"证书/解"映射为证明$G(R(x))=1$的解的方式来构造归约, 以确保此条件成立. 例如, 在{{ref:exe:dominatingsetex}}中, 我们构造图$H$时, 使得对于$G$中的每个顶点覆盖$S$, 集合$S \cup I$(其中$I$是孤立顶点)将是$H$中的一个支配集. 
    *   **可靠性**:  此条件是指, 如果$F(x)=0$则$G(R(x))=0$, 或者(取其逆否命题)如果$G(R(x))=1$则$F(x)=1$. 这有时是直接的, 但通常比完全性条件更难证明, 并且在更高级的归约中(例如{{ref:thm:isetnpc}}中$3\text{SAT} \leq_p \text{ISET}$的归约), 证明可靠性是分析的主要部分. 例如, 在{{ref:exe:dominatingsetex}}中, 为了证明可靠性, 我们需要证明对于图$H$中的**每一个**支配集$D$, 在图$G$中存在一个大小至多为$|D| - \ell$的顶点覆盖$S$(其中$\ell$是孤立顶点的数量). 这具有挑战性, 因为支配集$D$可能不一定是我们"心中所想"的那个. 具体来说, 在上面的证明中, 我们需要修改$D$以确保它不包含形如$w_{u,v}$的顶点, 并且重要的是要证明这个修改仍然保持$D$是一个支配集的性质, 同时不会使其变大. 

每当你需要提供一个归约时, 你应该确保你的描述包含所有这些部分. 虽然有时将归约的描述与其分析交织在一起很诱人, 但通常将两者分开, 并将分析分解为高效性、完全性和可靠性三个部分会更清晰. 

## 从独立集归约到最大割

我们现在要证明独立集问题可以归约到**最大割问题**(maximum cut problem), 这里将最大割问题建模为函数$\text{MAXCUT}$, 该函数在输入一对$(G, k)$时, 当且仅当$G$包含一个至少包含$k$条边的割时输出$1$. 由于两者都是图问题, 从独立集到最大割的归约会将一个图映射到另一个图, 但正如我们将要看到的, 输出图不一定与输入图具有相同的顶点或边. 

```admonish quote title=""
{{thmc}}{thm:isettomaxcut}[最大割问题的困难性]

$\text{ISET} \leq_p \text{MAXCUT}$
```

```admonish proof collapsible=true title="{{ref:thm:isettomaxcut}}的证明思路"
我们将把一个图$G$映射到一个图$H$, 使得$G$中的一个大的独立集成为$H$中割开许多边的划分. 我们可以将$H$中的一个割看作将每个顶点着色为"蓝色"或"红色". 我们将添加一个特殊的"源"顶点$s^*$, 将其连接到所有其他顶点, 并且不失一般性地假设它被着为蓝色. 因此, 我们着红色的顶点越多, 割开的来自$s^*$的边就越多. 现在, 对于原始图$G$中的每一条边$u,v$, 我们将添加一个特殊的"构件", 这是一个包含$u$、$v$、源点$s^*$和另外两个额外顶点的小子图. 我们设计这个构件的方式是, 如果红色顶点在$G$中不是独立集, 那么$H$中对应的割将被"惩罚", 即它不会割开那么多边. 一旦我们为自己设定了这个目标, 就不难找到一个能实现它的构件——参见下面的证明. 这里的**核心技巧**再次是使用(这次是稍微更巧妙一点的)构件. 
```

```admonish pic id="iset2maxcutoverviewfig"
![iset2maxcutoverviewfig](./images/chapter14/iset2maxcutoverview.png) 

{{pic}}{fig:iset2maxcutoverview} 在从$\text{ISET}$到$\text{MAXCUT}$的归约中, 我们将一个具有$n$个顶点、$m$条边的图$G$映射为具有$n+2m+1$个顶点和$n+5m$条边的图$H$, 方法如下. 图$H$包含一个特殊的"源"顶点$s^*$, $n$个顶点$v_0,\ldots,v_{n-1}$, 以及$2m$个顶点$e_0^0,e_0^1,\ldots,e_{m-1}^0,e_{m-1}^1$, 每一对对应$G$的一条边. 对于每个$i\in [n]$, 我们在$s^*$和$v_i$之间连一条边; 如果$G$的第$t$条边是$(v_i,v_j)$, 那么我们就添加五条边$(s^*,e_t^0),(s^*,e_t^1),(v_i,e_t^0),(v_j,e_t^1),(e_t^0,e_t^1)$. 意图是, 如果我们至多将$v_i,v_j$中的一个与$s^*$割开, 那么我们将能割开这五条边中的$4$条; 而如果我们将$v_i$和$v_j$都与$s^*$割开, 那么我们最多只能割开其中的三条. 
```

```admonish proof collapsible=true title="{{ref:thm:isettomaxcut}}的证明"
我们将一个具有$n$个顶点和$m$条边的图$G$转化为一个具有$n+1+2m$个顶点和$n+5m$条边的图$H$, 方式如下(另见{{ref:fig:iset2maxcutoverview}}). 图$H$包含$G$的所有顶点(但不包含它们之间的边!), 此外$H$还具有: 

* 一个特殊的顶点$s^*$, 它连接到$G$的所有顶点. 
* 对于每一条边$e=\{u,v\} \in E(G)$, 两个顶点$e_0,e_1$, 使得$e_0$连接到$u$, $e_1$连接到$v$, 并且我们将边$\{e_0,e_1 \},\{ e_0,s^* \},\{e_1,s^*\}$添加到$H$中. 

通过证明$G$包含一个大小至少为$k$的独立集当且仅当$H$有一个割开至少$k+4m$条边的割, 即可得出{{ref:thm:isettomaxcut}}. 我们现在证明这个等价关系的两个方向: 

**第1部分: 完备性**: 如果$I$是$G$中一个大小为$k$的独立集, 那么我们可以定义$S$为$H$中具有以下形式的割: 我们让$S$包含$I$的所有顶点, 并且对于$E(G)$中的每一条边$e=\{u,v \}$, 如果$u\in I$且$v\not\in I$, 那么我们将$e_1$添加到$S$; 如果$u\not\in I$且$v\in I$, 那么我们将$e_0$添加到$S$; 如果$u\not\in I$且$v\not\in I$, 那么我们将$e_0$和$e_1$都添加到$S$. (我们不需要担心$u$和$v$都在$I$中的情况, 因为它是一个独立集.)我们可以验证, 在所有情况下, 从$S$到其补集在对应于$e$的构件中的边数将是四条(参见{{ref:fig:ISETtoMAXCUT}}). 由于$s^*$不在$S$中, 我们还有$k$条从$s^*$到$I$的边, 总共$k+4m$条边. 

**第2部分: 可靠性**: 假设$S$是$H$中的一个割, 割开了至少$C=k+4m$条边. 我们可以假设$s^*$不在$S$中(否则我们可以将$S$"翻转"为其补集$\overline{S}$, 因为这不会改变割的大小). 现在让$I$是$S$中对应于$G$原始顶点的顶点集合. 如果$I$是一个大小为$k$的独立集, 那么我们就完成了. 情况可能并非总是如此, 但我们将看到, 如果$I$不是独立集, 那么它的大小也大于$k$. 具体来说, 我们定义$m_{in}=|E(I,I)|$为$G$中完全包含在$I$内的边的集合, 并令$m_{out}=m-m_{in}$(即, 如果$I$是一个独立集, 则$m_{in}=0$, $m_{out}=m$). 根据我们构件的性质, 我们知道对于$G$的每一条边$\{u,v\}$, 当$u$和$v$都在$S$中时, 我们最多能割三条边, 否则最多能割四条边. 因此, $S$割开的边数$C$满足$C \leq |I| + 3m_{in}+4m_{out} = |I|+ 3m_{in} + 4(m-m_{in})=|I|+4m-m_{in}$. 由于$C = k +4m$, 我们得到$|I|-m_{in} \geq k$. 现在, 我们可以通过遍历$I$内部的$m_{in}$条边中的每一条, 并从$I$中移除该边的一个端点, 从而将$I$转化为一个独立集$I'$. 得到的集合$I'$是图$G$中的一个大小为$|I|-m_{in} \geq k$的独立集, 从而完成了可靠性条件的证明. 
```

```admonish pic id="ISETtoMAXCUTfig"
![ISETtoMAXCUTfig](./images/chapter14/iset2maxcutgadgetanalysis.png) 

{{pic}}{fig:ISETtoMAXCUT} 在从独立集到最大割的归约中, 对于每个$t\in [m]$, 我们都有一个对应于原始图中第$t$条边$e= \{ v_i,v_j\}$的"构件". 如果我们将包含特殊源顶点$s^*$的割侧视为"白色", 另一侧视为"蓝色", 那么最左边和中间的图显示, 如果$v_i$和$v_j$不都是蓝色, 那么我们可以从构件中割开四条边. 相比之下, 通过枚举所有可能性可以验证, 如果$u$和$v$都是蓝色, 那么无论我们如何给中间顶点$e_t^0,e_t^1$着色, 我们都最多只能从构件中割开三条边. 上图仅包含构件的边, 忽略了连接$s^*$到顶点$v_0,\ldots,v_{n-1}$的边. 
```

```admonish pic id="isettomaxcutcodefig"
![isettomaxcutcodefig](./images/chapter14/is2maxcut.png) 

{{pic}}{fig:isettomaxcutcode} 从独立集到最大割的归约. 右侧是实现该归约的Python代码. 左侧是归约应用的一个示例输出, 我们将其应用于通过对3CNF公式$(x_0 \vee \overline{x}_3 \vee x_2) \wedge (\overline{x}_0 \vee x_1 \vee \overline{x}_2) \wedge (\overline{x}_1 \vee x_2 \vee x_3)$运行{{ref:thm:isetnpc}}的归约得到的独立集实例. 
```

## 从3SAT归约到最长路径

**注意**:  本节内容还有点凌乱; 可以跳过它, 或者只阅读而不深入证明细节. 证明出现在Sipser书籍的7.5节. 

计算机科学中最基本的算法之一是Dijkstra算法, 用于查找两个顶点之间的**最短路径**. 我们现在证明, 相比之下, **最长路径**问题的高效算法将意味着3SAT存在多项式时间算法. 

```admonish quote title=""
{{thmc}}{thm:longpaththm}[最长路径问题的困难性]

$$3\text{SAT} \leq_p \text{LONGPATH}$$
```

```admonish pic id="longpathfig"
![longpathfig](./images/chapter14/3sat_longest_path_red_without_path.png) 

{{pic}}{fig:longpath} 我们可以将一个3SAT公式$\varphi$转换为一个图$G$, 使得图$G$中的最长路径对应于$\varphi$中的一个可满足赋值. 在此图中, 黑色部分对应于$\varphi$的变量, 蓝色部分对应于子句. 一条足够长的路径将首先蜿蜒穿过黑色部分, 为每个变量选择"上部路径"(对应于赋值为`True`)或"下部路径"(对应于赋值为`False`). 然后, 为了达到最大长度, 该路径将穿过蓝色部分, 要在对应于诸如$x_{17} \vee \overline{x}_{32} \vee x_{57}$这样的子句的两个顶点之间穿行, 对应的顶点必须之前未被遍历过. 
```

```admonish pic id="longpathfigtwo"
![longpathfigtwo](./images/chapter14/3sat_to_longest_path_reduction.png) 

{{pic}}{fig:longpathtwo} 上图显示了标记了最长路径的图, 路径中对应于变量的部分为绿色, 对应于子句的部分为粉色. 
```

```admonish proof collapsible=true title="{{ref:thm:longpaththm}}的证明思路"
为了证明{{ref:thm:longpaththm}}, 需要展示如何将一个3CNF公式$\varphi$转换为一个图$G$和两个顶点$s,t$, 使得$G$有一条长度至少为$k$的路径当且仅当$\varphi$是可满足的. 归约的思路如{{ref:fig:longpath}}和{{ref:fig:longpathtwo}}所示. 我们将构造一个包含一条潜在的漫长"蜿蜒路径"的图, 该路径对应于公式中的所有变量. 我们将以只有当我们拥有可满足赋值时才能使用构件的方式, 为$\varphi$的每个子句添加一个"构件". 
```

```python
def TSAT2LONGPATH(φ):
    """将 3SAT 归约为 LONGPATH"""
    def var(v): # 返回变量以及指示是是否带有否定的True/False
        return int(v[2:]),False if v[0]=="¬" else int(v[1:]),True
    n = numvars(φ)
    clauses = getclauses(φ)
    m = len(clauses)
    G =Graph() 
    G.edge("start","start_0")
    for i in range(n): # 为每个变量添加2条长度为m的路径
        G.edge(f"start_{i}",f"v_{i}_{0}_T")
        G.edge(f"start_{i}",f"v_{i}_{0}_F")
        for j in range(m-1): 
            G.edge(f"v_{i}_{j}_T",f"v_{i}_{j+1}_T")
            G.edge(f"v_{i}_{j}_F",f"v_{i}_{j+1}_F")
        G.edge(f"v_{i}_{m-1}_T",f"end_{i}")
        G.edge(f"v_{i}_{m-1}_F",f"end_{i}")
        if i<n-1:
            G.edge(f"end_{i}",f"start_{i+1}")
    G.edge(f"end_{n-1}","start_clauses")
    for j,C in enumerate(clauses): # 为每个子句添加构件
        for v in enumerate(C):
            i,sign = var(v[1])
            s = "F" if sign else "T"
            G.edge(f"C_{j}_in",f"v_{i}_{j}_{s}")
            G.edge(f"v_{i}_{j}_{s}",f"C_{j}_out")
        if j<m-1:
            G.edge(f"C_{j}_out",f"C_{j+1}_in")
    G.edge("start_clauses","C_0_in")
    G.edge(f"C_{m-1}_out","end")
    return G, 1+n*(m+1)+1+2*m+1
```

```admonish proof collapsible=true title="{{ref:thm:longpaththm}}的证明"
我们构造一个从$s$蜿蜒到$t$的图$G$, 如下所示. 在$s$之后, 我们添加一系列$n$个长循环. 每个循环有一条"上部路径"和一条"下部路径". 一条简单路径不能同时走上部路径和下部路径, 因此它需要恰好选择其中一条才能从$s$到达$t$. 

我们的意图是, 图中的一条路径将对应于一个赋值$x\in \{0,1\}^n$, 其意义是: 在第$i$个循环中走上部路径对应于赋$x_i=1$, 走下部长路径对应于赋$x_i=0$. 当我们蜿蜒穿过所有对应于变量的$n$个循环到达$t$后, 我们需要穿过$m$个"障碍": 对于每个子句$j$, 我们将有一个由一对顶点$s_j,t_j$组成的小构件, 它们之间有两条路径. 例如, 如果第$j$个子句的形式为$x_{17} \vee \overline{x}_{55} \vee x_{72}$, 那么一条路径会穿过对应于$x_{17}$的下部循环中的一个顶点, 一条路径会穿过对应于$x_{55}$的上部循环中的一个顶点, 第三条路径会穿过对应于$x_{72}$的下部循环中的一个顶点. 我们看到, 如果我们在第一阶段按照一个可满足赋值走, 那么我们将能够找到一个空闲的顶点从$s_j$旅行到$t_j$. 我们将$t_1$链接到$s_2$, $t_2$链接到$s_3$, 等等, 并将$t_m$链接到$t$. 因此, 一个可满足赋值将对应于一条从$s$到$t$的路径, 该路径穿过每个对应于变量的循环中的一条路径, 以及每个对应于子句的循环中的一条路径. 我们可以使对应于变量的循环足够长, 以至于我们必须走完每个循环中的整条路径, 才有可能获得一条与可满足赋值相对应的那样长的路径. 但如果我们这样做, 那么能够到达$t$的唯一方式就是我们走过的路径对应于一个可满足赋值, 否则我们将有一个子句$j$, 在该子句中, 我们无法在不使用之前已用过的顶点的情况下从$s_j$到达$t_j$. 
```

```admonish pic id="threesattwolongpathfig"
![threesattwolongpathfig](./images/chapter14/3sat2longpath.png) 

{{pic}}{fig:threesattwolongpath} 将$3\text{SAT}$到$\text{LONGPATH}$的归约应用于公式$(x_0 \vee \neg x_3 \vee x_2) \wedge (\neg x_0 \vee x_1 \vee \neg x_2) \wedge (x_1 \vee x_2 \vee \neg x_3)$的结果. 
```

### 关系总结

我们已经证明存在若干函数$F$, 可对之证明形如“若$F\in \mathbf{P}$则$3\text{SAT} \in \mathbf{P}$”的命题. 因此, 即便仅为这些问题之一设计出多项式时间算法, 也将导致获得$3\text{SAT}$的多项式时间算法(参见{{ref:fig:reductiondiagram}}). 在[第15章](./chapter_15.md)中, 我们将对这些函数证明逆命题(“若$3\text{SAT} \in \mathbf{P}$则$F\in \mathbf{P}$”), 从而得出它们与$3\text{SAT}$具有**等价复杂性**(equivalent complexity)的结论. 

```admonish pic id="reductiondiagramfig"
![reductiondiagramfig](./images/chapter14/reduction_inc_diagram.png) 

{{pic}}{fig:reductiondiagram} 截至目前, 我们已证明$\mathbf{P} \subseteq \mathbf{EXP}$, 且多个重要问题(如$3\text{SAT}$与$MAXCUT$)属于$\mathbf{EXP}$, 但其是否属于$\mathbf{P}$尚未可知. 然而, 由于$3\text{SAT} \leq_p MAXCUT$, 可排除$MAXCUT \in \mathbf{P}$但$3\text{SAT} \not\in \mathbf{P}$的可能性. $\mathbf{P_{/poly}}$与$\mathbf{EXP}$类的关系尚不明确. 我们知道$\mathbf{EXP}$不包含$\mathbf{P_{/poly}}$, 因为后者甚至包含不可计算函数, 但不确定$\mathbf{EXP} \subseteq \mathbf{P_{/poly}}$是否成立(尽管普遍认为不成立, 特别是$3\text{SAT}$和$\text{MAXCUT}$被认为不属于$\mathbf{P_{/poly}}$). 
```

```admonish hint title="本章回顾"
* 众多看似无关的计算问题的计算复杂性可通过**归约**相互关联. 
* 若$F \leq_p G$, 则$G$的多项式时间算法可转化为$F$的多项式时间算法. 
* 等价而言, 若$F \leq_p G$且$F$**不**存在多项式时间算法, 则$G$亦不存在. 
* 我们已发展多种技术证明对于重要函数$F$有$3\text{SAT} \leq_p F$. 有时可利用归约的**传递性**: 若$3\text{SAT} \leq_p G$且$G \leq_p F$, 则$3\text{SAT} \leq_p F$. 
```

## 习题

## 参考书目 {#reductionsbibnotes }

学界定义了多种归约概念. {{ref:def:reduction-def}}所述概念常被称为**映射归约**(mapping reduction)、**多到一归约**(many to one reduction)或**Karp归约**. 

**极大独立集**(maximal independent set, 与**最大独立集**maximum independent set相对)是寻找独立集的“局部极大解”任务: 即一个无法再添加顶点而不破坏独立性的独立集$S$(此类集合称为**顶点覆盖**). 与寻找**最大独立集**不同, **极大独立集**可通过贪心算法高效求得, 但此类局部极大解可能远小于全局最大解. 

独立集到最大割的归约取材自[相关讲义](https://people.engr.ncsu.edu/mfms/Teaching/CSC505/wrap/Lectures/week14.pdf). 十二面体Hamiltonian路径图像由[Christoph Sommer](https://commons.wikimedia.org/wiki/File:Hamiltonian_path.svg)提供. 

我们曾提及算法设计所用归约与证明难解性所用归约间的界限有时是模糊的. **SAT求解器**领域(参见([Gomes, Kautz, Sabharwal, Selman, 2008](https://scholar.google.com/scholar?hl=en&q=Gomes,+Kautz,+Sabharwal,+Selman+Satisfiability+solvers)))是绝佳例证: 该领域研究者利用SAT算法(虽在最坏情况下需指数时间, 但在实际诸多实例中远快于此)结合形如$F \leq_p \text{SAT}$的归约, 为其他目标函数$F$设计算法. 