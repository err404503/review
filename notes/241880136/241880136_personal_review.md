# 241880136 Personal Review

> 论文：*How to Approximate A Set Without Knowing Its Size In Advance* (Pagh, Segev, Wieder, 2013)
> 日期：2026/7/28

---

## 一、论文概述

### 1.1 问题设定

本文研究的是动态近似成员查询问题的一个基本变体：当待表示集合的最终大小 $n$ 事先未知时，数据结构必须支持在线插入和成员查询，无假阴性，假阳性率 $\le\varepsilon$，并且空间尽可能接近信息论下界。

当 $n$ 已知的时候，已被证明空间占用是 $\Theta(n\log(1/\varepsilon))$ bits，单个元素占用 $\Theta(\log(1/\varepsilon))$ bits。当 $n$ 未知时，已有工程方案（如 Scalable Bloom Filter 等）空间为 $\Omega(n\log n)$——差了约一个 $\log n$ 因子。已有方案的问题在于：为了保证总假阳性 $\le\varepsilon$，旧方案让每个子过滤器的假阳性 $\varepsilon_i$ 随 $i$ 指数衰减，导致后段 $\log(1/\varepsilon_i) \approx i$，求和后附加项 $\Omega(n\log n)$。

### 1.2 本文结论

论文的核心贡献由三部分组成。其一，通过压缩论证证明了空间下界 $(1-o(1))n\log(1/\varepsilon)+\Omega(n\log\log n)$，首次揭示了"已知 $n$"与"未知 $n$"两种设定之间的超线性空间差距——每元素的 $\Omega(\log\log n)$ 额外开销无法消除。其二，提出构造一（几何增长数据结构），以 $\varepsilon_i=\Theta(\varepsilon/i^2)$ 的多项式衰减替代旧方案的指数衰减，将附加空间从 $\Omega(n\log n)$ 压至 $O(n\log\log n)$，代价是查询 $O(\log n)$。其三，提出构造二（单字典前缀扩展），用成对独立哈希和 $r=\lceil\log\log u\rceil$ 位辅助标签控制签名翻倍，将查询改进为 $O(1)$ 最坏情况，同时保持相同量级的空间上界；通过去摊销化进一步给出插入最坏 $O(1)$ 的版本。上下界在渐近阶上完全一致，$\Omega(n\log\log n)$ 是不可消除的固有代价。

---

## 二、学习过程

详细学习内容和过程在具体笔记内。

### Abstract & Introduction 阅读 (7.15–16)
阅读论文 §1。关键认识：当 $n$ 已知时，Bloom Filter 的空间为 $1.44n\log(1/\varepsilon)$，而信息论下界为 $n\log(1/\varepsilon)$；Carter et al. 和 Raman-Rao 字典法可将空间逼近到 $(1+o(1))n\log(1/\varepsilon)$。当 $n$ 未知时，已有的 Scalable Bloom Filter 等方案让 $\varepsilon_i$ 指数衰减，导致空间 $\Omega(n\log n)$。本文的核心贡献是证明了 $\Omega(n\log\log n)$ 的额外开销不可避免，并给出两个达到该下界的构造。形成笔记，含定理陈述及本文构造与 Bloom Filter 的属性对比表。

### 预备知识整理 (7.16)
阅读 §1.2 与 §2，整理 Word RAM 模型、$k$-wise 独立哈希族及四个概率工具（马尔可夫/切尔诺夫/联合界/信息论下界）的定义和它们在论文证明中的位置。阅读 §1.2 五类相关研究（Bloom Filter / Counting BF / 字典法 AM / 在线离线分离 / 动态完美哈希），与课件 Hashing.pdf 和 Sketching.pdf 对照。形成笔记。

### 下界证明学习 (7.16–17)
阅读 §3。参照 241880161 的整理笔记进行逐步验算。补全了切尔诺夫界 $\delta=5/4$ 的代数代入、引理 3.4 中 $|C_{j_1}| \approx \alpha n(\gamma-1)$ 满足 $\alpha \ge 1/\sqrt{n}$ 时 $\ge n^{1/2}$ 的验证、$n>6$ 时 $1/2-1/n\ge 1/3$ 的条件检查、以及 $j_1/j_2$ 定义中的保守放缩推导。

### 构造一 (7.17)

阅读论文 §4 和 RR03 原始论文摘要。

从 RR03 的信息论下界 $B=\lceil\lg\binom{m}{n}\rceil$ 出发，推导到论文的起点结论，再进一步完整推导出 $\sum|B_i| \to (1+o(1))n\log(1/\varepsilon)+O(n\log\log n)$，验证技术正确性。

### 构造二 (7.17, 7.22)

阅读论文 §5 。

- 将构造二的基础构建与可选扩展分开学习。
- 学习并补全三部分正确性证明：
  - 容量不溢出：早期/近期子序列，$\sum_{j=1}^{i-r} 2^{i-r-1} + \sum_{j=i-r+1}^i 2^j \le 2^{i+2}$
  - 无假阴性：Mode 2 迁移规则保证。
  - 假阳性率 $\le\varepsilon+u^{-c}$：碰撞 $\le 2^{i+2}\cdot 2^{-\ell_i} \le \varepsilon$。
- 从 RR03 公式 $\to u=2^{\ell_i}$ $\to$ $2^i$ 推导出最终空间结果
- 注意论文未显式讨论 $\bot$ 填充可能导致的边界效应——当 $i$ 很大时 $h(x)$ 比特不够，最右端标签位是 $\bot$，会使部分元素提前翻倍。论文在容量分析中假设近期子序列的标签非空，只影响最后 $\log\log u$ 个阶段且只改变常系数，尝试解决该问题
- 阅读两个可选拓展并推导其证明

### 拓展阅读与相关研究 (7.26)

- 检索引用本文的后续论文（Semantic Scholar），筛选出三篇。
- 下载并逐篇分析：研究对象、此前未解决的问题、主要结论、与本文的关系等
- 逐条完成大作业要求中内容，形成草稿

### review 撰写 (7.27以后)

- 重新整理合作者已完成的review初稿结构
- 修正和完善自己重点负责和初稿缺失的部分
- 多次修改证明流程追求信达雅

---

## 三、独立完成的推导和外延工作简述

### 1. RR03 字典信息论下界到单个 B_i 空间

阅读 RR03 原文摘要：紧凑动态字典的空间为 $B + o(B)$，其中 $B = \lceil\lg\binom{m}{n}\rceil$ 是区分 $\binom{m}{n}$ 种可能集合所需的信息论下界。由斯特林近似，$B \approx n\log(m/n) + O(n)$。论文 §1.2 的字典法将元素哈希到较小域 $[m]$ 后存入字典——取 $m = 2^i/\varepsilon_i$，$n = 2^i$，则 $m/n = 1/\varepsilon_i$，代入得：

$$|B_i| = (1+o(1)) \cdot 2^i \cdot \log(1/\varepsilon_i)$$

论文将此式作为黑盒引用，推导由本人补全。

### 2. 构造一总空间从求和到 $\max$ 放缩的完整推导

所有 $B_i$ 的空间求和：

$$\text{总空间} = \sum_{i=1}^{\lceil\log n\rceil} (1+o(1)) \cdot 2^i \cdot \log(1/\varepsilon_i)$$

使用 $\sum a_i b_i \le \max\{b_i\} \cdot \sum a_i$ 放缩（其中 $\sum 2^i \approx n$，最后一块约占总量一半，常数被 $(1+o(1))$ 吸收）：

$$\sum_i (1+o(1)) \cdot 2^i \cdot \log(1/\varepsilon_i) \le (1+o(1)) \cdot n \cdot \max_{1 \le i \le \lceil\log n\rceil}\{\log(1/\varepsilon_i)\}$$

代入 $\varepsilon_i = \Theta(\varepsilon/i^2)$：$\log(1/\varepsilon_i) = \log(1/\varepsilon) + 2\log i + O(1)$。$i \le \log n$，$\log(i^2)$ 在 $i = \lceil\log n\rceil$ 取最大值 $2\log\log n = O(\log\log n)$。最终 $\max_i\{\log(1/\varepsilon_i)\} = \log(1/\varepsilon) + O(\log\log n)$，总空间 $(1+o(1))n\log(1/\varepsilon) + O(n\log\log n)$。

### 3. $\varepsilon_i$ 多项式与指数衰减的本质区别

旧方案（Scalable Bloom Filter, ABP+07 等）用指数衰减：$\varepsilon_{i+1} \approx \varepsilon_i / 2$ → $\varepsilon_i = n^{-\Omega(i)}$ → $\log(1/\varepsilon_i) = \Omega(i)$ → $\sum 2^i \cdot \Omega(i) = \Omega(n\log n)$。论文用多项式衰减：$\varepsilon_i = \Theta(\varepsilon/i^2)$ → $\log(1/\varepsilon_i) = \log(1/\varepsilon) + 2\log i + O(1)$ → 求和附加项仅 $O(n\log\log n)$。构造一原创点在 $\varepsilon_i$ 的衰减速度选择。

### 4. 构造二容量不溢出证明的补全与放缩条件验证

论文 §5 将 $D_i$ 条目按来源子序列分为两类（$r = \lceil\log\log u\rceil$）：

**早期子序列（$S_1,\dots,S_{i-r}$）**：标签中 $r$ 位已耗尽（$\alpha_1 = \bot$），每次过渡翻倍。每个 $S_j$ 元素贡献 $2^{i-j-r}$ 个条目。论文原文给出求和 $\sum_{j=1}^{i-r} 2^{i-r-1}$。

**近期子序列（$S_{i-r+1},\dots,S_i$）**：标签尚有剩余（$\alpha_1 \neq \bot$），每次过渡仅扩展 1 位，每元素贡献恰好 1 条目。精确求和 $\sum_{j=i-r+1}^{i} 2^j = 2^{i+1} - 2^{i-r+1}$。

总条目数经放缩 $\le 2^{i+2}$。本人额外验证了放缩条件 $i \le 2^{r+2} + r$ 恒成立：$r = \lceil\log\log u\rceil$ → $2^{r+2} \ge 4\log u$ → $2^{r+2} + r \ge 4\log u + \log\log u > \log u \ge i$（因 $i \le \log n \le \log u$）。

### 5. 构造二无假阴性的显式论证

Mode 2 迁移规则中，对 $D_{i-1}$ 的每对 $(y, \alpha_1\cdots\alpha_r)$：若 $\alpha_1 \neq \bot$，延长为 $y\alpha_1$——跟随实际哈希值的下一位；若 $\alpha_1 = \bot$，同时保留 $y0$ 和 $y1$——覆盖该位的两种可能。两种情况均覆盖 $h_i(x)$ 的精确值，$x$ 在任意后续阶段都至少有 $(h_i(x), \cdot)$ 在 $D_i$ 中。

### 6. 构造二假阳性率的代数推导

$D_i$ 中至多 $2^{i+2}$ 个键。成对独立性保证 $h_i(x)$ 与任一已有键碰撞的概率 $\le 2^{i+2} \cdot 2^{-\ell_i}$。代入 $\ell_i = \lceil\log(1/\varepsilon)\rceil + i + 2$：

$$2^{i+2} \cdot 2^{-\ell_i} \le 2^{i+2} \cdot 2^{-(\log(1/\varepsilon) + i + 2)} = 2^{i+2} \cdot \frac{\varepsilon}{2^{i+2}} = \varepsilon$$

### 7. 构造二空间分析：从 RR03 到最终结果的四步推导

单桶内 $n$ 个元素，$n > u^\delta$，当前阶段 $2^{i-1} \le n \le 2^i-1$。

**(a)** RR03 空间保证：$(1+o(1)) \cdot n \cdot (\log(u/n) + r)$。

**(b)** 论文代入 $u = 2^{\ell_i}$（键是 $\ell_i$ 位签名，全集体积 $2^{\ell_i}$），以 $2^i$ 近似 $n$（相差不超过因子 2，损失被 $O(1)$ 吸收）：

$$|D_i| \le (1+o(1)) \cdot n \cdot (\log(2^{\ell_i}/2^i) + r) = (1+o(1)) \cdot n \cdot (\ell_i - i + r)$$

**(c)** 代入 $\ell_i = \lceil\log(1/\varepsilon)\rceil + i + 2$ → $\ell_i - i = \log(1/\varepsilon) + O(1)$。$r = \lceil\log\log u\rceil$，由 $n > u^\delta$ 保证 $\log\log u = \log\log n + O(1)$：

$$|D_i| \le (1+o(1)) \cdot n \cdot (\log(1/\varepsilon) + r + O(1)) = (1+o(1))n\log(1/\varepsilon) + O(n\log\log n)$$

**(d)** 桶化后总空间 $u^{\delta/2} \cdot s_{\max}$，均衡保证 $s_{\max}$ 为上述中 $n$ 替换为单桶容量 $(1+o(1))n/u^{\delta/2}$ 的结果。

### 8. 三种可选扩展的学习

**去摊销化**：将 Mode 2 的 $O(2^i)$ 初始化工作量均摊到该子序列的 $2^i$ 次插入，每次额外投入常数步 → 插入最坏 $O(1)$。代价是不能用桶化（过渡不再限于一个桶），空间升至 $O(n\log(1/\varepsilon) + n\log\log n)$。

**删除支持**：论文放弃多重集思路（早期签名副本太多无法精准删除），改为主辅字典方案。辅助字典精确存假阳性元素（零假阳性率），主字典条目额外存删除指示位。删除时计算最小字典序签名（原始签名补零至当前长度）并标记。$h(x)$ 到最小签名高概率单射，碰撞概率同假阳性率。

**空间分析补充**：假阳性率可假定 $\le 1/\log u$（已允许 $O(n\log\log n)$ 冗余），期望假阳性数 $O(n/\log u)$，辅助字典期望 $O(n)$ 位。多项式哈希控制高概率界到期望常数倍。

### 9. 后续论文：检索、筛选与逐篇分析

通过 Semantic Scholar 检索引用本文的论文（以 arXiv ID 1304.1188 查询），从 20 条返回结果中筛选出与"未知 $n$ 设定下近似成员/紧凑字典"直接相关的三篇，逐一下载 PDF 并完成独立分析：

**(a) Aleph Filter（Dayan, Bercea, Pagh, PVLDB 2024）**。可无限扩展的过滤器，基于 InfiniFilter，所有操作常数时间。Pag 是本文第一作者也是此文的合著者。§7 Related Work 明确引用本文下界并声称 Widening 和 Predictive 模式均达到该下界。与本文关系：同一问题，Aleph 是本文理论结果的工程实现。

**(b) Resizable Retrieval（Kuszmaul et al., 2026）**。解决 Demaine 2006 开放问题——将检索结构空间上界从预设 $N$ 替换为当前 $n$。技术路线直接延续本文的"碰撞风险分摊"（§1 明确引用 [PSW13]），引入 midyard 处理收缩碰撞。与本文关系：使用本文技术。已下载 PDF。

**(c) Dynamic "Succincter"（Li et al., FOCS 2023）**。动态 aB-tree 仅需 3 bit 冗余，$O(\log n/\log\log n)$ 时间。参考文献中将本文列为紧凑字典领域工作之一。与本文关系：概念启发。

每篇按统一提纲（研究对象 / 此前未解决的问题 / 主要结论 / 核心方法 / 与本文关系 / 类比失效 / 区分原文与推断）撰写分析笔记后，摘要纳入 review §6.3。

---

## 四、review 撰写与统稿中的主要工作

### 1. 各节补全

**(a) 与 Bloom Filter 的对比表（§1.3）**。将 Bloom Filter 与三构造并列对比，覆盖共 9 个维度。来自 Abstract\&Introduction 阅读笔记。

**(b) 问题为什么重要（§1.2）**。初版仅 7 行。从笔记提取六场景重写——网络流量/LSM-tree/集合调和/入侵检测/CDN。

**(c) 技术的非凡之处（§1.5）**。撰写五条：超线性差距首次严格证明、紧致匹配、构造二三维最优、$g_i$ 标签的临界点设计、去摊销实时保证。

**(d) 构造二正确性与空间分析（§5.3–5.5）**。三性质完整证明（含类型划分 + 两个求和 + 放缩代数）+ 空间四步推导（RR03 → $u=2^{\ell_i}$ → $2^i$ 近似 $n$ → 桶化抵消）。含定理 5.1 陈述与证明概要。

**(e) 发表后的新研究（§6.3）**。三年独立分析文件的摘要写入，每篇含主要结论、核心方法与本文关系。

**(f) 概述与总结讨论（§0, §7）**。§0 学术摘要式概述与 §7 两次叙述。修正了旧版的一些错误。

### 2. 结构重排

改为使用三章统一模板，直觉 → 思路 → 详细推导，将直觉拆分迁移至 §3.1/§4.1/§5.1。

