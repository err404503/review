# 框架阅读

## 传统框架

- 布隆计数器
- 集合的最终元素数量n确定，采用流式插入
- 约束：

1. 无假阴性：属于集合的元素查询一定会返回存在
2. 假阳性上界$`\epsilon`$：不在集合中的元素误判率不超过$`\epsilon`$


- 理论空间占用：$`\Theta(nlog(\frac{1}{\epsilon}))`$ bits， 单一元素占用 $`\Theta(log(\frac{1}{\epsilon}))`$ bits


## 新框架

- 最终集合大小n不定，需要考虑结构自适应扩容，且要维持n尽可能小。
- 因此，传统最优空间理论不再适用，试证明额外付出$`\Omega(nloglogn)`$bits空间开销

## 下界定理（原文1.1）

- 在未知n的前提下，任意满足约束的近似成员结构存储空间的$`lower\ bound`$为$`(1-o(1))nlog(\frac{1}{\epsilon})+\Omega(nloglogn)`$。
- 需要注意的是，相比于传统的已知规模的情景，这里的每元素平均开销多出了$`\Omega(loglogn)`$bits，这意味着开销会随着集合规模增长，所以无法维持常数单元素存储。
- 下界的证明思路：
  1. **随机化转确定性**：利用平均论证，固定随机种子得到一个确定性结构$`B_{r^*}`$，使得对至少一半的插入序列，其假阳性集合测度$`\mu(\hat{S}_r)\leq 4\epsilon`$
  2. **压缩论证**：用$`B_{r^*}`$来编码序列本身——将序列$`S`$分成若干子序列$`C_i`$（$`|C_i|=\gamma^i`$），利用数据结构的状态来压缩编码$`C_i`$中不属于$`\hat{S}_{i-1}`$的元素
  3. 由于编码长度不能短于信息论下界，推得空间下界$`\beta \geq (1-\frac{1}{\gamma})\cdot log(1/\epsilon) + (1-9\epsilon)loglog_\gamma(1/\alpha) - O(1)`$
  4. 取$`\alpha=1/\sqrt{n},\ \gamma=2^{(logn)^\eta}`$得到最终下界$`(1-o(1))nlog(1/\epsilon)+\Omega(nloglogn)`$


## 上界定理（原文1.2）
- 存在一种数据结构，在未知集合大小n的情况下，使用$`(1+o(1))nlog(1/\epsilon)+O(nloglogn)`$ bits空间，即可满足假阳性率$`\epsilon`$。
- 论文给出了两种构造方案：

### 构造1：几何增长数据结构

**核心思想：**
- 将插入序列视为连续子序列，第i个子序列包含$`2^i`$个元素
- 为每个子序列$i$分配一个动态近似成员数据结构$`B_i`$，假阳性率$`\epsilon_i=\Theta(\epsilon/i^2)`$
- 查询时依次询问所有$`B_i`$，任一返回Yes则判定存在

**性能：**
- 空间：$`(1+o(1))nlog(1/\epsilon)+O(nloglogn)`$ bits
- 插入：期望摊还常数时间
- 查询：$`O(logn)`$时间（需遍历所有$B_i$）

**优点：** 实现简单直观
**缺点：** 查询时间随$`n`$对数增长，因为不知道元素属于哪个子序列

### 构造2：常数时间操作

**核心思想：**
- 任何时候只维护一个动态字典$`D_i`$（精确集合），存储当前已插入元素的某个超集
- 使用两两独立哈希函数$`h: U \to \{0,1\}^\ell$，其中$\ell \geq \lceil log(1/\epsilon) \rceil + log u + 2`$

**细节：**
- 同样将插入序列分为子序列$`s_i`$（$2^i$个元素）
- 处理第i个子序列时，对于每个元素$x$，在$D_i$中存储$(h_i(x), g_i(x))$：
  - $`h_i(x)`$：$`h(x)`$的最左$`\ell_i = \lceil log(1/\epsilon)\rceil + i + 2`$位（作为key）
  - $`g_i(x)`$：$`h(x)`$的下$`r = \lceil loglog u \rceil`$位（补$`\perp`$填充）
- 当进入下一个子序列时，将$`D_i`$中的每个$`(\text{key}, \alpha)`$展开迁移到$`D_{i+1}`$：
  - 若$`\alpha_1 \neq \perp`$：插入$`(key \cdot \alpha_1, \alpha_2\cdots\alpha_r\perp)`$
  - 若$`\alpha_1 = \perp`$：插入$`(key \cdot 0, \alpha)`$和$`(key \cdot 1, \alpha)`$
- 这种迁移方式保证了每个$`D_i`$只存储$`O(2^i)`$个值，从而匹配最优空间

**性能：**
- 空间：$`(1+o(1))nlog(1/\epsilon)+O(nloglogn)`$ bits
- 插入：期望摊还常数时间
- 查询：常数时间
- 可通过去摊还（de-amortization）将插入也优化为最坏情况常数时间，空间升至$`O(nlog(1/\epsilon)+nloglogn)`$ bits

**降低空间的桶化技巧：**
- 用哈希将元素分摊到$`u^{\delta/2}`$个桶中
- 各桶的数据结构按字交错存储，保证过渡操作最多只在一个桶中发生
- 额外空间只与单个桶大小成比例，而非总元素数

## 删除操作

- 无法检测是否尝试删除假阳性元素，因此要求用户保证"只删除确实在集合中的元素"
- 基本思路：用辅助字典处理签名冲突
  - 插入时，假阳性元素放入辅助结构（零假阳性率）
  - 删除时先尝试从辅助结构删除，否则从主结构中删除签名
- 为防止误删导致的假阴性，每个签名存储原始长度标记，删除时标记最小签名为"已删除"
- 后台进程清理已删除签名的空间

## 未来研究方向

- 理论：收紧$`\Omega(nloglogn)`$附加项的常数因子
- 实践：第一个构造查询时间为$`O(logn)`$较慢，第二个构造隐藏常数较大
- 期待设计同时满足最优空间和常数时间的实用数据结构，匹配本文空间下界

# 预备知识
### 单位代价Word RAM模型
- 模型设定：全集大小为$`u`$，单个元素占用一个字，字长$`\omega=\lceil log\ u \rceil bits`$ 
- 常数操作时间：对该模型种的元素进行加减、按位的布尔运算，任意位数的移位，整数乘法均为单位耗时
- 作为数据结构上下界证明的通用标准模型，用于进行空间、时间复杂度分析
- 我们将Word RAM模型作为统一分析基准，更多是因为下界证明不关心操作时间，只在意存储空间比特数

### k重独立哈希函数族
- 标准的k-独立哈希定义：
	哈希函数$`H:U\rightarrow V`$ 满足：任取k个互不相同的元素$`x_1,...,x_k`$，任意目标取值$`y_1,...y_k \in V`$，有：
	$$`Pr_{h\leftarrow H} [h(x_1)=y_1 \wedge h(x_2)=y_2\ \wedge ... 
	\wedge h(x_k)=y_k]=\frac{1}{|V|^k}`$$
- k重$`\delta`$ 近似独立哈希
	放宽约束：$`(h(x_1),h(x_2),...,h(x_k))`$与均匀分布的统计距离不超过$`\delta`$，允许微小偏差。
	我们期望使用该弱化哈希以降低实现开销。

### 马尔可夫不等式
- 设非负随机变量$`X \geq 0`$，对任意常数$`a>0`$，$`Pr[X \geq a] \leq \frac{E[X]}{a}`$ 
- 非负随机变量取远大于期望的值的概率存在上界；
- 求取该上界时只需要期望，无需方差与分布函数
> 论文中在3.1节证明：
- 随机结构下，$\mu(\hat{S}_r)$是非负随机变量，已知期望$`E[\mu(\hat{S}_r)]\leq 2\epsilon`$，取$`a = 4\epsilon`$ ，代入得到：$`Pr[\mu(\hat{S}_r)\geq 4\epsilon] \leq \frac{2\epsilon}{4\epsilon} = \frac{1}{2}`$ 
- 我们可以推导出存在随机种子$`r^*`$ ，至少一半的序列满足$`\mu(\hat{S}_{r^*})\leq 4\epsilon`$，即我们可以完成随机结构向确定性结构的归约。 

### 切尔诺夫界
- $`X = X_1 + X_2 + ... + X_m, X_j独立0-1伯努利随机变量, \mu = E[X]`$
- 对任意$`\delta > 0`$， $`Pr[X \geq (1 + \delta)\mu] \leq exp(-\frac{\delta^2\mu}{2+\delta})`$ 
- 独立伯努利变量和偏离期望的概率指数衰减，放缩精度远高于马尔可夫不等式。
> 论文3.2节中
- 分段$`C_i`$内元素均匀随机采样，$`k = |C_i \cap \hat{S}_{i-1}|`$ 是独立命中事件之和，期望$`E[k]\leq 4\epsilon|C_i|`$
- 取$`(1+\delta)\mu = 9\epsilon|C_i|`$，可得$`\delta`$为常数，尾概率$`exp(-|C_i|)`$，当$`|C_i|=n^{\Omega(1)}`$时趋近于0；
- 因此我们可以证明绝大多数序列是满足$`k\leq9\epsilon|C_i|`$的。

### 联合界
- 对任意可数事件$`A_1,A_2,A_3,..., Pr[\bigcup_i A_i]\leq \Sigma_iPr[A_i]`$ 
> 论文中
- 构造1中利用此建立多子过滤器总假阳性控制：$`\Sigma \epsilon_i \leq \epsilon`$
- 引理3.4中：对所有分段取并，重叠超标序列的总概率极小

### 无损编码信息论下界
- 若存在M条互不相同的序列，采用无损前缀编码存储所有序列，则至少需要$`log_2 M`$bits
> 论文3.2节中
- 合法序列集合$`S_{r^*,\epsilon}`$规模$`|S_{r^*,\epsilon}| \geq u^n / 3`$，因此编码该集合内任意序列，编码总长度必须满足：总编码比特数$`\geq log(u^n / 3)`$
- 假设数据结构存储空间仅为$`\beta m`$比特，若我们构造出一种编码方案，总编码长度严格小于$`log(u^n / 3)`$，则这与信息论下界矛盾。
# 3.1
### 证明目标
给定空间约束，推出单个元素平均占用比特$`\beta`$的下界

我们认为n是充分大的，满足$`n \leq \epsilon u`$。
对于区间参数$`\alpha`$，我们设定$`\frac{1}{\sqrt{n}} \leq \alpha < 1`$。
对于分段基数$`\gamma`$，其应当是大于等于2的任意整数。

我们不妨假设对任意插入长度m满足$`\alpha n < m < n`$，数据结构D至多占用$`\beta m`$比特空间。

那么我们期望能够获得$`\beta \geq (1-\frac{1}{\gamma}) · (log\frac{1}{\epsilon} + (1-9\epsilon)log_\gamma \frac{1}{\alpha} - \Theta(1))`$

换言之，如果数据结构D在集合规模位于$`(\alpha n, n)`$区间内，想要做到平均每个元素仅仅占用$`\beta`$比特，那么$`\beta`$一定会有下界。

### 推论1
我们不妨考虑$`\alpha = 1`$时，这就将命题退化到了传统的问题下。
在这种情况下，不等式将会退化成：
$`\beta \geq (1-\frac{1}{\gamma}(log\frac{1}{\epsilon} - \Theta(1))`$。
令$`\gamma \rightarrow \infty`$，可得$`\beta \geq (1-o(1))log\frac{1}{\epsilon}`$，即平均开销下界为$`log\frac{1}{\epsilon}`$，符合我们传统下界的判断。
因此，我们认为该定理的自洽性无问题。

### 推论2
考虑$`\alpha = \frac{1}{\sqrt{n}}, \gamma = 2^{(logn)^\eta},其中\eta \in (0,1)`$
此时的总存储空间下界：
$`(1-o(1))n log\frac{1}{\epsilon}+\Omega(nloglogn)`$
说明未知规模的情况下，$`\Omega(nloglonn)`$的开销是无法避免的

### 3.1
我们希望将随机结构转化为确定性结构
我们定义：
- 随即比特串r：数据结构内部所有哈希、随机选择以来的只读随机源，存储空间不计入结构开销；
- 确定性结构$`B_r`$：我们将随机源r固定后、整个插入、查询流程将不再存在任何随机行为，因此输入将完全决定输出
- 判定超集$`\hat{S_r}`$：对于插入序列S，所有被$`B_r`$查询返回“存在”的元素构成的集合。
- 对于任意插入序列S，一定会满足$`S \subseteq \hat{S_r}`$。

#### 归一化测度
- $`\mu(\hat{S_r})= \frac{|\hat{S_r}|}{u}`$ 
- 代表判断超集在全局全集U中的占比，等价于随机抽取一个元素被误判的概率
#### 合法序列集合
- $`S_{r,\epsilon} = \{S \in U^n | \mu(\hat{S_r} \leq 4\epsilon)\}`$
- 集合内所有插入序列都应当满足：固定随机种子r后，判定超集的规模可控，假阳性的概率应当不高于$`4\epsilon`$

- 存在随机串$`r^*`$，使得合法序列数量满足$`|S_{r^*,\epsilon}| \geq \frac{u^n}{2}`$
1. 计算$`E[\mu(\hat{S_r})]`$
	- 由于原始随机结构要求假阳性概率$`\leq \epsilon`$，因此对于任意长度n的插入序列S：
	- $`E_{r\leftarrow\{0,1\}^*}[\mu (\hat{S_r})] \leq \epsilon + \frac{n}{u}`$
	- 文中限定$`n \leq \epsilon u`$，因此$`\frac{n}{u} \leq \epsilon`$，代入：
	- 右式 $`\leq 2\epsilon`$
2. 使用马尔可夫不等式放缩
	- $`\mu(\hat{S_r}) \geq 0`$ 为非负随机变量，取阈值$`a = 4\epsilon`$：
	- $`Pr[\mu(\hat{S_r}) \geq 4\epsilon] \leq \frac{E[\mu(\hat{S_r})]}{4\epsilon} \leq \frac{1}{2}`$
3. 计数推导存在$`r^*`$
	- 这个是显然的。
	- 将全部$`u^n`$种插入序列中，至少会有一半的序列降落在$`S_{r^*,\epsilon}`$种，所以得证。
# 3.2
### 分段
- 在3.1章节中，我们已经得到了确定性结构$`B_{r^*}`$与$`合法序列集合S=S_{r^*,\epsilon}`$，对任意长度为n的序列S，我们进行分段处理：
1. 我们定义分段基数：$`\gamma \geq 2`$（这是由引理3.1得到的自定义参数，我们期望人为设定一个分段长度的增长速度）
2. 第i段的子序列$`C_i`$：其长度应当严格=$`\gamma^i`$
3. 前缀序列$`S_i`$：前i段全部元素拼接得到，则其总长度$`n_i = \Sigma_{j=1}^i \gamma^j`$
4. 超集单调性不变式：
	$`\hat{S_1} \subseteq \hat{S_2} \subseteq ... \subseteq \hat{S_i} \subseteq \hat{S_{i+1}...}`$
	每次插入新元素都只能扩大判定超集，因此测度单调不减：
	$`0 \leq \mu (\hat{S_1}) \leq \mu(\hat{S_2}) \leq ... \leq 4\epsilon`$
5. 为什么要进行分段处理：
	将完整的插入流程拆分为指数增长的片段，从中截取一段$`C_i`$作为编码论证的分析单元。以通过平均分配总测度增量得到单段增量上界。

### 引理3.3
- 命题：对于任意的S，存在下标i满足前缀长度$`n_i \in [\alpha n,n]`$，且超集测度增量满足$`\mu(\hat{S_i}) - \mu(\hat{S_{i-1}}) \leq \frac{4\epsilon}{log_\gamma(1/\alpha)-2}`$ 
- 尝试证明：
	1. 划定区间的上下界$`j_1,j_2`$，保证所有$`i \in [j_1,j_2]`$对应的前缀长度$`n_i`$落在$`(\alpha n, n)`$内
	2. 总测度增量上限为$`4\epsilon`$，区间内共有$`j_2 - j_1`$个分段增量
	3. 由抽屉原理可知，由于总增量之和不超过$`4\epsilon`$，则必然存在某段的增量不超过平均值
	4. $`j_2 - j_1 \geq log_\gamma(1/\alpha) - 2`$，代入即可

### 引理3.4
- 命题：满足$`k(S) \leq 9\epsilon|C_i|`$的序列S至少占集合总量的$`\frac{u^n}{3}`$
- 证明：
	 1. 我们假设序列逐段均匀独立从全集U采样
	 2. 对于重叠元素，我们期望$C_i$均匀随机，$`E[k]=|C_i|·\mu(\hat{S_{i-1}}) \leq 4\epsilon |C_i|`$
	 3. 利用切尔诺夫界进行放缩：取$`(1+\delta)·4\epsilon = 9\epsilon`$，则可以得到常数$`\delta`$，尾概率$`Pr[k \geq 9\epsilon|C_i|] \leq exp(-|C_i|)`$
	 4. 规模条件$`|C_i| = n^{\Omega(1)}`$，单段超标的概率指数应当极小；
	 5. 使用联合界来处理所有分段，结合引理3.2给出的基数$`|S| \geq \frac{u^n}{2}`$，最终可以推导出满足$`k \leq 9\epsilon |C_i|`$的序列应当不少于$`u^n / 3`$ 


### 无损编码
#### 四段式无损编码完整构造
- 对于任意的合法序列S，我们设计一套无损编码，解码器应当仅依靠编码串就能完整还原S
- 对于每个编码，我们将其划分为4个部分：
	1. 下标i：开销为log log n比特
	2. 编码部分除了$`C_i`$以外的全部元素
		- 需要将序列中检测到不属于$`C_i`$的元素原始完整存储，开销：$`(n - c_i) log u`$比特
		- 解码器应当能够识别所有非$`C_i`$的元素，仅剩余$`C_i`$段需要依靠数据结构状态做压缩编码
	3. 完整存储当前数据结构内部的状态
		- 通过$`b_i`$比特的开销，我们可以直接保存结构内部的所有比特
		- 解码器通过$`b_i`$比特就可以直接还原$`\hat{S_{i-1}}和\hat{S_i}`$这两个超集，以此实现$`C_i`$段压缩编码。
	4. 分段$`C_i`$的相对压缩编码
		$`C_i`$中的元素可以分为两类，我们需要进行分开编码：
		1) 重叠元素：
			- k个元素落在旧超集$`\hat{S_{i-1}}`$内
			- 其编码开销为$`k·log(\mu(\hat{S_i})·u)+O(1)`$
			- 由于该元素属于旧超集，因此我们仅需在规模为$`\mu(\hat{S_i})u`$的集合内定位即可
		2) 新增的超级元素：
			- 即可以假设$`c_i - k`$个元素落在了$`\hat{S_i}\setminus \hat{S_{i-1}}`$ 内
			- 对于该类的编码开销，应当为$`(c_i -k)·log(\Delta \mu · u)+O(1)`$
			- 由于该类元素仅存在于本次新增的超集区间中，因此候选空间更小，我们期望以更短的编码进行构造
至此，我们得到了总编码长度：
$`TotalLen = loglogn+(n-c_i)logu+b_i+(c_i-k)log(\Delta\mu·u)+klog(\mu(\hat{S_i})u)+O(c_i))`$
但上述公式过于复杂，不适合我们实际使用

### 化简
1. 对于公式中的对数项，我们进行以下拆分
	$`log(\Delta\mu·u)=logu+log\Delta\mu`$
	$`log(\mu(\hat{S_i})u)=logu+log\mu(\hat{S_i})`$
	代入原公式并提取公共的log u项
	$`TotalLen = b_i + loglogn + nlogu +(c_i-k)log\Delta\mu + klog\mu(\hat{S_i})+O(c_i)`$
2. 由引理3.3可知：$`log\Delta\mu \leq log\epsilon log_\gamma(1 / \alpha) + O(1)`$
	又，我们知道$`\mu(\hat{S_i}) \leq 4\epsilon \leq \epsilon`$
	因此$`log \mu(\hat{S_i}) \leq log\epsilon`$
	又$`k \leq 9\epsilon c_i`$
	将两项对数全部放缩后合并所有包含$`c_i`$的项可得：
	$`(c_i - k)log \Delta\mu + klog\mu(\hat{S_i}) \leq c_i(log\epsilon - (1-9\epsilon)log_\gamma (1/\alpha) + O(1))`$
因此我们可以得到简化后的总编码长度
$`TotalLen \leq b_i + loglogn + nlogu + c_i(log\epsilon - (1-9\epsilon)log_\gamma (1/\alpha) + O(1))`$
### 建立矛盾不等式
- 无损编码的核心约束：能够区分$`|S|\geq u^n / 3`$条的不同序列，因此编码的长度必须满足
- $`TotalLen \geq log(u^n /3) = nlogu-O(1)`$
- 将化简后的上界代入不等式：
- $`b_i + loglogn + nlogu +c_i(log\epsilon - (1-9\epsilon)log_\gamma(1/\alpha)+O(1))\geq nlogu - O(1)`$
- 最终可以得到核心空间下界：
- $`b_i \geq c_i(log\frac{1}{\epsilon}+(1-9\epsilon)log_\gamma\frac{1}{\alpha}-O(1)`$

# 定理3.1
### 命题
设D是全集大小为u，假阳性上限为$`\epsilon`$的未知规模近似成员查询数据结构。
我们认为n是充分大的，满足$`n \leq \epsilon u`$。
对于区间参数$`\alpha`$，我们设定$`\frac{1}{\sqrt{n}} \leq \alpha < 1`$。

设对任意插入长度m满足$`\alpha n < m < n`$，数据结构D至多占用$`\beta m`$比特空间。则对于任意的整数$`\gamma \geq 2`$ ，我们期望能够获得
$`\beta \geq (1-\frac{1}{\gamma}) · (log\frac{1}{\epsilon} + (1-9\epsilon)log_\gamma \frac{1}{\alpha} - \Theta(1))`$
### 证明
- 通过前文3.2章节我们通过压缩编码建立了矛盾不等式，进而得到了单段空间下界：
- $`b_i \geq c_i(log\frac{1}{\epsilon}+(1-9\epsilon)log_\gamma\frac{1}{\alpha}-O(1)`$
1. 建立$`c_i和n_i`$的数量关系
	- 对等比数列求和：
	-  $`n_i = \gamma + \gamma^2 + ... + \gamma^i = \gamma · \frac{\gamma^i-1}{\gamma-1}=c_i · \frac{\gamma-1/\gamma^{i-1}}{\gamma-1}`$
	- 当$`i \rightarrow \infty`$时，$`1/\gamma^{i-1} \rightarrow 0`$，因此：
	- $`n_i = c_i · \frac{\gamma}{\gamma-1}·(1-o(1)) \Longrightarrow c_i = n_i · \frac{\gamma-1}{\gamma}·(1+o(1))`$
	- 因此我们可以简单地认为：$`c_i = \Theta(n_i)`$
2. 带入我们的反证假设前提$`b_i \leq \beta n_i`$
	- 将$`c_i`$代入下界并化简后可得：
	- $`\beta \geq (1-\frac{1}{\gamma})·(log\frac{1}{\epsilon}+(1-9\epsilon)log_\gamma\frac{1}{\alpha}-\Theta(1))`$
	- 证毕
3. 验证推论1
	- $`\alpha`$ = 1时结论是显然的
4. 推导主定理1.1
	- 考虑$`\alpha = \frac{1}{\sqrt{n}}, \gamma = 2^{(logn)^\eta},其中\eta \in (0,1)`$
	- 先行计算对数项：
	- $`log_\gamma\frac{1}{\alpha} = log_{2^{(log n)^\eta}}\sqrt{n} = \frac{\frac{1}{2} logn}{(log n)^\eta} = \frac{1}{2}(log n)^{1-\eta} = \Theta(loglogn)`$
	- 将其回代到定理3.1的结论中，总空间$`S=\beta n`$
	- $`S \geq (1-o(1))nlog\frac{1}{\epsilon}+\Omega(nloglogn)`$

**至此，我们已经完成了第3章空间下界的全部数学推导**
过程：
1. 原始随机近似结构 → 马尔可夫不等式（引理 3.2）→ 固定r∗转为确定性结构
2. 对插入序列指数分段$`C_i,S_i`$​ → 抽屉原理（引理 3.3）：单段超集增量可控
3. 均匀采样 + 切尔诺夫界（引理 3.4）：绝大多数分段重叠元素数量可控
4. 构造四段无损编码 → 信息论编码下界建立矛盾不等式
5. 化简得到单段$`b_i`$​下界 → 结合$`c_i,n_i`$​等比关系证明定理 3.1
6. 代入两组参数 → 分别得到已知 / 未知规模空间下界（定理 1.1）
# 横向对比
对于下界，我们已经有了较为准确的定义：
$`(1-o(1))nlog\frac{1}{\epsilon}+\Omega(nloglogn)`$

但对于上界，我们仍然有两套方案：
1. 多几何扩容过滤器
	- 空间开销：$`(1+o(1))nlog\frac{1}{\epsilon}+O(nloglogn)`$
	- 插入期望为均摊常数
	- 查询开销为$`O(logn)`$
2. 高性能单字典构造
	- 空间开销：$`(1+o(1))nlog\frac{1}{\epsilon}+O(nloglogn)`$
	- 插入期望为均摊常数
	- 查询为最坏常数
	- 但是去除均摊后空间为$`O(nlog\frac{1}{\epsilon}+nloglogn)`$

**结论**
下界为$`\Omega(nlog\frac{1}{\epsilon}+nloglogn)`$，上界为$`O(nlog\frac{1}{\epsilon}+nloglogn)`$，两者渐近阶完全一致。说明未知规模带来的loglogn的平均开销是不可避免的，也因此不存在更低阶的数据结构

在过往的方案中（比如Scalable Bloom Filter），为了保证总假阳性$`\leq \epsilon`$，每一级子过滤器的假阳性$`\epsilon_i`$需要按照$`\frac{1}{i^2}`$衰减；每级都要存储独立的哈希函数，累加后总空间开销会达到$`\Omega(nlogn)`$，从而并非当前的最优解。

#### loglog n的意义
1. 证明了已知n和未知n这两种模型确实存在超线性空间差距
2. log log n本身增长极其缓慢，相比于旧方案的log n更具优势
3. 首次给出了自适应扩容结构的固有理论存储下界，为工程实现提供了理论证明。

# Total Review
### 阶段 1：问题区分

1. 标准近似成员（已知 n）：预先给定集合大小，空间下界$`\Theta(nlog1/\epsilon)`$，单元素常数比特；
2. 新问题（未知 n）：流式插入、无法预知最终规模，要求自适应扩容；
3. 核心直觉：近似结构必须维护单调扩张判定超集$`\hat{S}`$，插入会强制超集大幅扩张，带来额外存储开销。

### 阶段 2：前置储备

1. 符号体系：u全集大小、n元素总数、ϵ假阳性、$`\hat{S}`$判定超集、$`\mu = |\hat{S}|/u`$测度；
2. 计算模型：Word RAM，全文空间统计单位为比特；
3. 四大数学工具及用途：
    - 马尔可夫不等式：引理 3.2，筛选优质随机种子；
    - 切尔诺夫界：引理 3.4，控制分段重叠元素；
    - 联合界：合并多事件概率；
    - 无损编码下界：压缩论证核心，编码比特≥log∣M∣。

### 阶段 3：定理 3.1 命题复述

给定充分大$`n \leq \epsilon u`$，$1\sqrt{n} \leq \alpha < 1$，$`\gamma \geq 2`$；假设所有$`m \in (\alpha n, n)`$序列占用空间$`\beta ,`$比特，则：

$`\beta \geq (1-\frac{1}{\gamma})·(log\frac{1}{\epsilon}+(1-9\epsilon)log_\gamma\frac{1}{\alpha}-\Theta(1))`$

### 阶段 4：完整证明链路

1. **3.1 随机结构→确定性结构（引理 3.2）** 固定随机串r得到$`B_r`$​，满足$`S \subseteq \hat{S_r}`$​；利用假阳性约束算出$`E[\mu(\hat{S_r})] \leq 2\epsilon`$；马尔可夫不等式得$`Pr[\mu \geq 4\epsilon] \leq 1/2`$，存在$`r^*`$使得至少一半序列满足$`\mu(\hat{S_{r^*}}) \leq 4\epsilon`$，只需分析$`B_{r^*}`$。
2. **3.2 分段前置两条引理**
    - 分段规则：序列拆分为长度$`\gamma^i`$的$`C_i`$，前缀$`S_i`$​，$`\hat{S_i}`$​单调不减；
    - 引理 3.3（抽屉原理）：存在i满足$`n_i \in [\alpha n,n]`$，单次测度增度$`\delta \mu \leq \frac{4\epsilon}{log_\gamma(1/\alpha)-2}`$；
    - 引理 3.4（切尔诺夫界）：分段重叠元素$`k \leq 9\epsilon|C_i|`$的序列占绝大多数。
3. **核心压缩编码矛盾论证** 构造四段无损编码：下标i、非$`C_i`$原始元素、数据结构状态$`b_i、C_i`$​​相对编码；拆分重叠 / 新增元素编码长度，代入$`\delta \mu、k`$上界化简；结合信息论编码下界建立不等式，消元移项得到单段空间下界： $`b_i \geq c_i · (log\frac{1}{\epsilon}+(1-9\epsilon)log_\gamma\frac{1}{\alpha}-\Theta(1))`$
4. **完成定理 3.1 证明** 等比数列求和得$`c_i = \Theta(n_i)`$，代入反证假设$`b_i \leq \beta n_i`$，两边除以$`n_i`$得到β下界。
5. **参数赋值导出主定理 1.1**
    - $`\alpha = 1`$：退化至已知规模，$`\beta \geq (1-o(1))log(1/\epsilon)`$；
    - $`\alpha = 1/\sqrt{n}, \gamma = 2^{(log n)^\eta}`$：化简$`log_\gamma(1/\alpha)=\Theta(loglogn)`$，总空间下界：$`(1-o(1))nlog\frac{1}{\epsilon}+\Omega(nloglogn)`$

### 阶段 5：紧性验证

两套构造上界均为$`(1+o(1))nlog(1/\epsilon)+O(nloglogn)`$，下界与上界渐近同阶，证明$`\Omega(nloglogn)`$是不可消除的固有开销。
