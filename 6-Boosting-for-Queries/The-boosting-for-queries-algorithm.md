# 查询算法的Boosting

我们将使用第[2](/2-Basic-Terms/Overview.html)节中概述的数据库的*行表示*，其中我们将数据库视为行的多重集或$$\mathcal{X}$$的元素。固定数据库大小$$n$$、数据域$$\mathcal{X}$$和敏感度最大为$$\rho$$的实值查询的查询集$$\mathcal{Q}={q:\mathcal{X}^*→\mathbb{R}}$$。

我们假设存在一个基本大纲生成器（在第[6.2](/6-Boosting-for-Queries/Base-synopsis-generators/A-generalization-bound.html)节中，我们将看到如何构造这些）。接下来，我们需要的基本生成器的性质是，对于查询集$$\mathcal{Q}$$上的任何分布$$\mathcal{D}$$，基本生成器的输出都可以用于计算大部分查询的准确答案，其中 “大分数” 是根据$$\mathcal{D}$$给出的权重定义的。基生成器由$$k$$参数化，即要采样的查询数；$$\lambda$$是输出的精度要求；$$\eta$$是对“大”的度量，描述了大部分查询的含义，$$\beta$$是失败概率。

**定义 6.1**（$$(k,\lambda,\eta,\beta)-$$基本大纲生成器）对于固定的数据库大小$n$，数据域$$\mathcal{X}$$和查询集$$\mathcal{Q}$$，考虑大纲生成器$$\mathcal{M}$$，其独立地从$$\mathcal{Q}$$上的分布$$\mathcal{D}$$对$$k$$个查询进行采样并输出大纲。我们称$$\mathcal{M}$$是一个$$(k,\lambda,\eta,\beta)-$$基本大纲生成器，如果对于任何$$\mathcal{Q}$$上的分布$$\mathcal{D}$$，除了$$\beta$$概率之外，所有$$\mathcal{M}$$的硬币翻转，$$\mathcal{M}$$输出的大纲$$\mathcal{S}$$对于由$$\mathcal{D}$$加权的$$\mathcal{Q}$$的$$1/2+\eta$$质量分数是$$\lambda-$$精确的：
$$
\begin{aligned}
   \underset{q\sim\mathcal{D}}{\text{Pr}}[|q(\mathcal{S})-q(x)|\leq\lambda]\geq1/2+\eta.\qquad \qquad (6.1)
\end{aligned}
$$
查询增强算法可用于任何类别的查询和任何不同的私有基本大纲生成器。 运行时间继承自基本大纲生成器。Booster在$$|\mathcal{Q}|$$中投入了准线性的额外时间，特别是其运行时间并不直接依赖于数据域的大小。

要指定提升（boosting）算法，我们需要指定停止条件、聚合机制以及用于更新$$\mathcal{Q}$$上的当前分布的算法。

**停止条件.** 我们将固定$$T$$轮运行算法——这将是我们的停止条件。选择$$T$$以保证高的准确度（以极大可能）；正如我们将看到的，只需要$$\log|\mathcal{Q}|/\eta^2$$轮就足够了。

**更新分布.** 尽管分布从未在输出中直接显示，但基本大纲$$\mathcal{A}_1,\mathcal{A}_2,\mathcal{A}_3,...,\mathcal{A}_T$$会被揭示，并且在构造$$\mathcal{A}_i$$时，每个$$\mathcal{A}_i$$在原则上都能从$$\mathcal{D}_i$$上泄露所选查询信息。因此，我们需要限制在相邻数据库上获得的概率分布之间的最大散度。这在技术上是具有挑战性的，因为给定$$\mathcal{A}_i$$，数据库在构建$$\mathcal{D}_{i+1}$$时涉及到的非常多。

初始分布$$\mathcal{D}_1$$在$$\mathcal{Q}$$上是均匀分布的。一个更新$$\mathcal{D}_t$$的标准方法是增加处理不良元素的权重，在我们的情况下，对于$$|q(x)-q(A_t)|>\lambda$$的查询，通过一个固定的因子（增加处理不良元素的权重），比如$$e$$，并通过相同的因子减少处理良好的元素权重。（然后将权重归一化以便总和为 1。）为了困难化，令$$x=y\cup\{\xi\}$$，并假设当数据库是$$y$$时，所有的查询$$q$$都被$$\mathcal{A}_t$$很好的处理，但是加上$$\xi$$后导致处理失败，例如，一个1/10的查询；即，对于所有查询$$q$$，$$|q(y)-	q(\mathcal{A}_t)|\leq\lambda$$，但是对于某些$$|\mathcal{Q}|/10$$的查询，$$|q(y)-	q(\mathcal{A}_t)|\geq\lambda$$。请注意，因为$$\mathcal{A}_t$$甚至在数据库是$$x$$时都有9/10的查询“表现良好”，无论$$x,y$$中的哪一个是真实的数据集，它都可以从基础sanitizer返回。我们关心的是更新的影响：当数据库为$$y$$时，所有查询都得到了很好的处理，并且没有重新加权（规范化之后），但是当数据库为$$x$$时，有一个重新加权：十分之一的查询的权重增加了 ，剩下的十分之九的权重减少了。 这种重新加权的差异可以在下一次迭代中通过$$\mathcal{A}_{t+1}$$检测到，这是可观察的，并且将根据数据库是$$x$$还是$$y$$，从相当不同的分布中抽取样本构建。

例如，假设我们从均匀分布$$\mathcal{D}_1$$开始。那么$$\mathcal{D}_2^{(y)}=\mathcal{D}_1^{(y)}$$，其中$$\mathcal{D}_i^{(z)}$$表示当数据库是$$z$$时的第$$i$$轮分布。这是因为每个查询的权重都减少了一个因子$$e$$，这在归一化中消失了。因此每一个$$q\in\mathcal{Q}$$在$$\mathcal{D}_2^{(y)}$$中被分配了权重$$1/\mathcal{Q}$$。相反，当数据库为$$x$$时，“不满意”的查询具有归一化权重
$$
\frac{\frac{e}{|\mathcal{Q}|}}{\frac{9}{10}\frac{1}{|\mathcal{Q}|}\frac{1}{e}+\frac{1}{10}\frac{e}{|\mathcal{Q}|}}.
$$
考虑任意这样不满意的查询$$q$$。比例$$\mathcal{D}_2^{(x)}(q)/\mathcal{D}_2^{(y)}(q)$$由下式给出
$$
\begin{aligned}
\frac{\mathcal{D}_2^{(x)}(q)}{\mathcal{D}_2^{(y)}(q)}&=\frac{\frac{\frac{e}{|\mathcal{Q}|}}{\frac{9}{10}\frac{1}      {|\mathcal{Q}|}\frac{1}{e}+\frac{1}{10}\frac{e}{|\mathcal{Q}|}}}{\frac{1}{\mathcal{|Q|}}}\\
&=\frac{10}{1+\frac{9}{e^2}}\overset{def}{=}F\approx4.5085.
\end{aligned}
$$
现在，$$\ln F \approx 1.506$$，即使基本生成器在第2轮中使用的查询选择没有明确公开，它们可能可以从公开的结果$$\mathcal{A}_2$$中检测到。因此，每个查询存在高达1.506的潜在隐私损失（当然，我们期望取消；我们只是试图解释困难的根源）。通过确保基本生成器所使用的样本数量相对较小，可以部分地解决这一问题，尽管我们仍然存在这样的问题，即在多次迭代中，即使在相邻数据库上，分布$$\mathcal{D}_t$$也可能非常不同地演化。

解决方案是去减弱重新加权过程。我们不是总是使用固定比例来增加权重（当回答是“精确”时）或减少权重（当回答为“不精确”时），而是为“精确性”（$$\lambda$$）和“不精确性”（$$\lambda+\mu$$分别设置阈值，以便适当的选择与基本生成器输出的位大小成比例的$$\mu$$；见引理[6.5](/6-Boosting-for-Queries/Base-synopsis-generators/A-generalization-bound.html)）。误差低于或高于这些阈值的查询的权重通过因子$$e$$分别降低或增加了。对于误差介于这两个阈值之间的查询，我们线性缩放权重变化的自然对数：$$1-2(|q(x)-q(\mathcal{A}_t)|-\lambda)/\mu$$，因此误差幅度超过$$\lambda+\mu/2$$的查询增加权重，而误差幅度小于$$\lambda+\mu/2$$的查询减少权重。

减弱的缩放减少了任何个体对任何查询的重新加权的影响。这是因为个体只能影响查询的真实答案——因此也只能影响基本大纲生成器的输出$$q(\mathcal{A}_t)$$——通过一个很小的量，而且减弱将该量除以一个参数$$\mu$$，该参数将被选择用于补偿从执行Boosting算法过程中获得的$$T$$分布中选择的（全部）$$kT$$样本。这有助于确保隐私。直观地，我们将这些$$kT$$样本中的每一个视为 “微型机制”。我们首先在任何一轮限制采样的隐私损失（声明[6.4](/6-Boosting-for-Queries/The-boosting-for-queries-algorithm.html)），然后通过组合定理限制累积损失。

“准确”和“不准确”阈值之间的差距（$$\mu$$）越大，每个个体对查询权重的影响就越小。这意味着更大的差距对隐私更好。然而，为了准确性，大的差距是不好的。如果不准确阈值很大，我们只能保证基本大纲生成器在非常不精确的查询下，重新加权期间其权重会大幅增加。这降低了boosting算法的准确性保证：误差大致等于“不精确”阈值$$(\lambda+\mu)$$。

**聚合.** 对于$$t\in[T]$$，我们将运行基本生成器去获得一个大纲$$\mathcal{A}_t$$。大纲将通过取中位数进行聚合：给定$$\mathcal{A}_1,...,\mathcal{A}_T$$，量$$q(x)$$是通过取使用每个$$\mathcal{A}_i$$计算的$$q(x)$$的$$T$$个近似值来估计的，然后计算他们的中位数。通过这种聚合方法，我们可以通过论证大多数$$\mathcal{A}_i$$，$$1\leq i\leq T$$为查询$$q$$提供$$\lambda+\mu$$（或更好）的精确度来展示$$q$$的精确度。这意味着$$q(x)$$的$$T$$近似值的中值将在真实值的$$\lambda+\mu$$范围内。

**注释.**

1. 在整个算法的运行过程中，我们跟踪几个变量（显式或隐式）。由$$q\in\mathcal{Q}$$索引的变量在查询集中保存与查询$$q$$有关的信息。由$$t\in[Q]$$索引的变量，通常在第$$t$$轮被计算，将会被用来构造用于在时间段$$t+1$$中采样的分布$$\mathcal{D}_{t+1}$$。

2. 对于谓词$$P$$，如果谓词为真，我们使用$$[[P]]$$表示1，如果它为假，则表示为0。

3. 算法中使用了最终的调整参数$$\alpha$$。它将被选择（见下面的推论[6.3](/6-Boosting-for-Queries/The-boosting-for-queries-algorithm.html)）取值为：
   $$
   \alpha=\alpha(\eta)=(1/2)\ln(\frac{1+2\eta}{1-2\eta}).
   $$

该算法如图6.1所示。步骤2（2b）中的量$$u_{t,q}$$是查询的新的、未归一化的权重。现在，让我们设置$$\alpha=1$$（这样我们就可以忽略任何$$\alpha$$因子）。设$$a_{j,q}$$为$$j$$轮中权重变化的自然对数，$$1\leq j\leq t$$，则新的权重由：
$$
u_{t,q}\leftarrow \exp(-\sum_{j=1}^ta_{j,q}).
$$
给定。因此，在上一步结束的前一步时，未归一化的权重为$$u_{t-1,q}=\exp(-\sum_{j=1}^{t-1}a_{j,q})$$并且更新对应于乘以$$e^{-a_{j,t}}$$。当和$$\sum_{j=1}^ta_{j,q}$$很大时，权重就很小。每当一个大纲给出一个非常好的$$q(x)$$的近似时，我们就在这个和上加1； 如果近似值只是中等好 （在$$\lambda$$和$$\lambda+\mu/2$$之间），我们加一个正数，但小于1。 相反，当大纲非常糟糕（差于$$\lambda+\mu$$精度)时，我们减去1； 当它勉强可以接受时(在$$\lambda+\mu/2$$和$$\lambda+\mu$$之间)，我们减去较小的量 。

![Figure6.1: Boosting for queries](/6-Boosting-for-Queries/img/figure61.jpg)

在下面的定理中，我们可以看到由$$\varepsilon_{sample}$$捕获的采样引起的隐私损失与精确和不精确阈值之间的差距之间的反比关系。 

**定理 6.1.** 令$$\mathcal{Q}$$是一个敏感度至多为$$ρ$$的查询族。对于适当的参数设置，并且在$$T=\log|\mathcal{Q}|/\eta^2$$轮次，图6.1的算法是一个精确且差分私有的查询增强（query-boosting）算法：

1. 当用基于$$(k, \lambda, \eta, \beta)$$的大纲生成器实例化时，Boosting算法的输出以至少$$1-T\beta$$的概率对$$\mathcal{Q}$$中的所有查询给出$$(\lambda+\mu)$$精确的回答，其中：
   $$
   \begin{aligned}
   \mu\in O(((\log^{3/2}|Q|)\sqrt k\sqrt{\log(1/\beta)}\rho)/(\varepsilon_{sample}\cdot\eta^3)).\qquad \qquad(6.2)
   \end{aligned}
   $$

2. 如果基本大纲生成器是$$(\varepsilon_{base},\delta_{base})-$$差分隐私，则boosting算法是$$(\varepsilon_{sample}+T\cdot\varepsilon_{base},\delta_{sample}+T\delta_{base})-$$差分隐私的

允许常数$$\eta$$被归入进大O符号，为简单起见取$$\rho=1$$，我们得到$$\mu=O(((\log^{3/2}|\mathcal{Q}|)\sqrt k\sqrt{\log(1/\beta)})/\varepsilon_{sample})$$。因此，我们看到减少基本清理程序（sanitizer）所需的输入查询数量$$k$$提高了输出的质量。 同样，从定理的完整陈述中，我们看到提高基本清理程序的泛化能力，这对应于具有更大的$$\eta$$值（更大的“强多数”），也提高了准确性。

【定理6.1的证明】我们首先证明准确性，然后证明隐私。

我们引入符号$$a_{t,q}^-$$和$$a_{t,q}^+$$，满足：

1. $$a_{t,q}^-,a_{t,q}^+\in\{-1,1\}$$；和
2. $$a_{t,q}^-\leq a_{t,q}\leq a_{t,q}^+$$。

回顾一下，更大的$$a_{t,q}$$表示对$$q(x)$$的大纲$$\mathcal{A}_t$$的更高质量的近似。

1. 如果$$\mathcal{A}_t$$在$$q$$上是$$\lambda-$$精确的，则$$a_{t,q}^-$$是1，否则是-1。要检查$$a_{t,q}^-\leq a_{t,q}$$，请注意如果$$a_{t,q}^{-}=1$$，则$$\mathcal{A}_t$$对$$q$$是$$\lambda-$$精确的，因此根据定义$$a_{t,q}=1$$。相反，如果我们有$$a_{t,q}^{-}=-1$$，那么因此我们总是有$$a_{t,q}\in[-1,1]$$。

   我们将使用$$a_{t,q}^-$$来降低基本生成器输出质量的下界。根据基本生成器的保证，对于$$\mathcal{D}_t$$质量的至少$$1/2+\eta$$部分，$$\mathcal{A}_t$$是$$\lambda-$$精确的。因此，
   $$
   \begin{aligned}
   r_t\overset{\Delta}{=}\sum_{q\in\mathcal{Q}}\mathcal{D}_t[q]\cdot a_{t,q}^-\geq(1/2+\eta)-(1/2-\eta)=2\eta.\qquad \qquad(6.3)
   \end{aligned}
   $$

2. 如果$$\mathcal{A}_t$$在$$q$$上是$$(\lambda+\mu)-$$精确的，$$a_{t,q}^+$$是-1，否则是1。要检查$$a_{t,q}\leq a_{t,q}^+$$，请注意如果$$a_{t,q}^{+}=-1$$，则$$\mathcal{A}_t$$对$$q$$是$$(\lambda+\mu)-$$不精确的，因此根据定义$$a_{t,q}=-1$$。相反，如果我们有$$a_{t,q}^{+}=1$$，那么因此我们总是有$$a_{t,q}\in[-1,1]$$。

   因此，$$a_{t,q}^+$$是正数当且仅当$$\mathcal{A}_t$$对于$$q$$至少是最小充分精确的。我们将使用$$a_{t,q}^+$$去证明聚合的精确度。当我们对$$a_{t,q}^+$$的值求和时，当且仅当大多数的$$\mathcal{A}_t$$提供可通过的（即，在$$\lambda+\mu$$内）对$$q(x)$$的近似时，我们才会得到一个正数。在这种情况下，中值将在$$\lambda+\mu$$内。

**引理 6.2.** 经过$$T$$轮提升后，除了$$T\beta$$的概率下，除了$$exp(-\eta^{2}T)$$大小的查询部分之外的所有答案都是$$(\lambda+\mu)$$-精确的。

【证明】 在最后一轮提升中，我们有：
   $$
   \begin{aligned}
      \mathcal{D}_{T+1}[q]=u_{T,q}/Z_T \qquad \qquad(6.4)
   \end{aligned}
   $$
由于$$a_{t,q} \leq a_{t,q}^+$$，我们有：
   $$
   \begin{aligned}
      u_{T,q}^+\overset{\Delta}{=}e^{-\alpha\sum_{t=1}^{T}a_{t,q}^+} \leq e^{-\alpha\sum_{t=1}^{T}a_{t,q}} = u_{T,q} \qquad \qquad(6.5)
   \end{aligned}
   $$

（上标“+”提醒我们这个未加权的值是使用术语$$a_{t,q}^+$$计算的。）注意，我们总是有$$u_{T,q}^+ \geq 0$$。结合方程(6.4)和(6.5)，对于所有$$q \in Q$$：
$$
   \begin{aligned}
      \mathcal{D}_{T+1}[q] \geq u_{T,q}^+/Z_T \qquad \qquad(6.6)
   \end{aligned}
$$
回想一下，$$\llbracket P \rrbracket$$表示当且仅当谓词$$P$$为真时值为1的布尔变量，我们转而检查值$$\llbracket\mathcal{A}对q来说是(\lambda+\mu)-不准确的\rrbracket$$。如果该谓词为1，则大多数$${\mathcal{A_{j}}}_{j=1}^T$$的情况一定是$$(\lambda+\mu)-不准确的$$，否则它们的中值将是$$(\lambda+\mu)-准确的$$。

根据我们对$$\sum_{t=1}^{T}a_{t,q}^+$$符号意义的讨论，我们有：
$$
\begin{aligned}
\mathcal{A}对q来说是(\lambda+\mu)-不准确的 &\Rightarrow \sum_{t=1}^{T}a_{t,q}^+ \leq 0\\
&\Leftrightarrow e^{-\alpha\sum_{t=1}^{T}a_{t,q}^+} \geq 1\\
&\Leftrightarrow u_{T,q}^+ \geq 1
\end{aligned}
$$
由于$$u_{T,q}^+ \geq 0$$，我们的结论是：
$$
\begin{aligned}
\llbracket\mathcal{A}对q来说是(\lambda+\mu)-不准确的\rrbracket \leq u_{T,q}^+
\end{aligned}
$$
将其与方程(6.6)一起使用可得出：
$$
\begin{aligned}
\frac{1}{|\mathcal{Q}|} \cdot \sum_{q \in \mathcal{Q}} \llbracket \mathcal{A}对q来说是(\lambda+\mu)-不准确的 \rrbracket &\leq \frac{1}{|\mathcal{Q}|} \cdot \sum_{q \in \mathcal{Q}}u_{T,q}^+\\
&\leq \frac{1}{|\mathcal{Q}|} \cdot \sum_{q \in \mathcal{Q}}\mathcal{D}_{T+1}[q]\cdotZ_T
&=Z_T/|\mathcal{Q}|.
\end{aligned}
$$
因此，以下断言完成了证明：
**断言6.3.**  经过$$t$$轮提升后，除了$$t\beta$$的概率下：
$$
\begin{aligned}
Z_t \leq exp(-\eta^2\cdot t) \cdot |\mathcal{Q}|
\end{aligned}
$$
【证明】根据基本概要生成器的定义，除了$$\beta$$概率外，生成的概要对于分布$$\mathcal{D}_t$$质量的至少$$(1/2 + \eta)$$部分来说是$$\lambda$$-精确的。回想一下，为 1 当且仅当 At 在 q 上是 λ 精确的，并且 a− t,q ≤ at,q 并进一步回想数量 rt , Σ q∈Q Dt[q]·a− t,q 定义在方程（6.3）中。如上所述，rt 衡量第 t 轮基本概要生成器的“成功”，其中“成功”是指更严格的 λ 准确度概念。正如方程（6.3）所总结的，如果以 λ 精度计算 Dt 质量的 (1/2 + η) 分数，则 rt ≥ 2η。现在还观察对于 t ∈ [T ]，假设基本消毒剂在第 t 轮中没有失败：
