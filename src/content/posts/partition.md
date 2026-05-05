---
title: VLSI 电路划分
published: 2026-05-06
description: 展示 Firefly 主题对 KaTeX 数学公式的支持，包括行内公式、块级公式和复杂数学符号。
---

## 贪心算法

### Kernighan-Lin 算法

#### 问题建模

KL 算法解决的是带平衡约束的图二划分问题，是 VLSI 电路划分的经典问题：
1.  **输入**：一个无向加权图 $G =(V, E)$
    - 顶点集 $V$ 的大小为 $|V|= 2n$（顶点总数是偶数，方便均分）
    - 边集 $E$ 的大小为 $|E|= m$
    - 边权重 $c_{AB}$ 表示顶点 $A$ 和 $B$ 之间的连接强度
2.  **输出**：将顶点集划分为两个子集 $X$ 和 $Y$，满足两个核心条件：
    - $X$ 和 $Y$ 之间隔代价最小。
    - $X$ 和 $Y$ 的顶点数量相等（各含 $n$ 个顶点）。
3.  **问题复杂度**：这是一个 **NP 难问题**，没有多项式时间的最优解，因此 KL 算法是一种启发式近似算法，用来快速找到一个足够好的解。

如果采用暴力搜索，穷举所有可能的二等分组合，复杂度为：$C_{2n}^n = \frac{(2n)!}{(n!)^2} = n^{O(n)}$

#### 算法流程

​	KL 算法是一个迭代式局部优化算法，主要采用一种贪心的策略，每次选择交换收益最大的点对进行交换，直到点对的交换不再产生任何的收益，主要流程如下：
1.  从一个初始的平衡划分出发；
2.  遍历所有未移动的定点对，找到一对交换收益最大的点，临时交换并进行标记；
3.  找到收益最大的前 `k` 次交换序列，整体执行点对的交换；
4.  重复迭代直到没有收益提升为止。

```
Algorithm 1: Kernighan-Lin 算法
Data: 图 G, 顶点集合 |V| = 2n 和普通边集合 E
Result: 割代价最小的一种划分 X, X'

1  随机划分得到一个均衡的初始解 X, X';
2  初始化移动过的顶点集合 M ← ∅, 最大收益 G = 0;
3  while G ≥ 0 do
4      gain_sum ← 0;
5      创建划分检查点;
6      for i = 1 to n do
7          gain_max ← -∞;
8          foreach x₁ ∈ X do
9              if x₁ ∈ M then
10                 continue;
11             foreach x₂ ∈ X' do
12                 if x₂ ∈ M then
13                     continue;
14                 if 交换 x₁, x₂ 的收益 gain(x₁, x₂) > gain_max then
15                     gain_max ← gain(x₁, x₂);
16                     记下 x₁, x₂ 顶点对;
17         执行 x₁, x₂ 点对的交换;
18         M ← M ∪ {x₁, x₂};
19         gain_sum ← gain_sum + gain_max;
20         记最大的 gain_sum 为 G 和对应的顶点对序列 ℒ;
21     M ← ∅;
22     回到检查点, 执行 ℒ 中所有的顶点对的交换;

Output: 划分 X, X'
```

​	每一轮迭代的复杂度是 $O(n^3)$（遍历所有顶点对 + $n$ 步交换），一些较好的实现可以把单步迭代时间复杂度降低为 $O(n^2 \log n)$。

#### KL 算法收益策略

用 $D_A$ 表示移动顶点 $A$ 后，割代价的变化量。

- $D_A = E_A - I_A$，外联边权重减去内联边权重。
- 交换 $(A,B)$ 的实际收益为 $gain(A,B) = D_A + D_B -2c_{AB}$

例如：

<img src="https://img.whitegoose.dpdns.org/image/屏幕截图 2026-05-04 145634.png" width="50%" height="50%" alt="图片描述" align="center" />

$D_A = 2-1 = 1$、$D_B = 1-1=0$、$gain(A,B) = D_A + D_B - 2c_{AB}$

需要平衡交换的数量，因此每次需要一对分别属于两个划分的顶点交换位置。这是 KL 算法的核心。

#### KL 算法的不足

1. 只能处理均匀权重的图
2. 只能做二等分的划分
3. 不支持超图
4. 寻找最优割代价收益的时间复杂度过高

---

### Fiduccia-Mattheyses 算法

FM 算法是 KL 算法的变种，能够解决上述 KL 算法的四个不足之处。

#### 问题建模

1. 输入：一个超图

   - 顶点集 $V$（且 $|V|=n$）

   - 超边集 $E$（连接顶点数量 $p$）

   - $V$ 中每一个顶点 $u$ 有点权重 $a_u$

   - $E$ 中每一个超边 $e$ 有边权重 $c_e$

   - 面积比例 $r$

2. 输出：两个划分集合满足

   - **总的超边割代价最小**

   - $\text{area}(X)/(\text{area}(X) + \text{area}(Y)) \ge r$

   - $\text{area}(Y)/(\text{area}(X) + \text{area}(Y)) \ge r$

这个问题也是 NP 困难的问题

与 KL 算法的不同之处：

- 不需要同时移动两个顶点，允许一步移动一个顶点
- 割代价收益更新使用桶排序的高效数据结构

#### 算法流程

FM 算法的核心是基于割代价收益的单节点移动：

1. 计算每个顶点的割代价收益
2. 所有顶点依据收益进行桶排序：使用桶（Bucket）结构，按收益值将顶点分组，所有收益相同的节点，都被放进同一个桶里。
3. 选择收益最大的顶点移动：同时检查移动是否满足面积比例约束。
4. 移动后的顶点标记为锁定状态：本次迭代中，已移动的节点不再参与后续收益计算，避免重复移动。
5. 更新相关顶点的收益并重新桶排序：节点移动后，会影响其邻居节点的收益，需重新计算并更新桶结构。
6. 重复迭代，直到所有节点被锁定：一轮迭代结束后，选择收益最大的移动序列执行，再进入下一轮迭代。

伪代码如下：

```
Algorithm 2: Fiduccia-Mattheyses 算法
Data: 图 G, 顶点集合 V 和普通边集合 E, 比例系数 r, 初始划分 X, X'
Result: 当前割代价最小的一种划分 X, X'

初始化移动过的顶点集合 M ← ∅;
移动的总收益 gain_sum ← 0;
while |M| < |V| do
    L ← {gain(i) | i ∈ V & i ∉ M};
    while 未执行移动操作 do
        x ← max{L};
        if r ≤ area(X)/(area(X) + area(X')) ≤ 1 - r then
            执行 x 的移动;
            M ← M ∪ {x};
            gain_sum ← gain_sum + gain(x);
            记下最大的 gain_sum 和对应的顶点序列 T;
        L ← L - x;
移动 T 中的顶点;

Output: 划分 X, X'
```

#### 割代价桶数据结构

- 初始化一系列桶，每个桶代表不同的割代价收益。

- 不同顶点计算自身的割代价，然后归类到合适的桶内。

- 在搜索最大割代价收益时，从最大收益处开始搜索，而不再需要依次搜索比较，可以显著加快最大收益的检索速度。

<img src="https://img.whitegoose.dpdns.org/image/屏幕截图 2026-05-04 161921.png" width="50%" height="50%" alt="图片描述" align="center" />

1.  **桶数组（左侧）**
    每个桶对应一个固定的收益值，从上到下依次是：
    - 正割代价收益桶
    - 最大割代价收益桶（当前最高收益的节点所在的桶）
    - 负割代价收益桶
2.  **节点链表（中间）**
    每个桶下面挂着一个链表，链表上的每个节点（`Cell #`）代表一个电路单元（顶点）。
    - 所有收益值相同的顶点，都会被挂在同一个桶的链表上。
3.  **顶点索引（下方）**
    每个顶点都有自己的索引（1, 2, ..., n），通过这个索引可以快速定位顶点的当前位置，方便后续更新收益时，把它从旧桶移到新桶。

#### 割代价收益

<img src="https://img.whitegoose.dpdns.org/image/屏幕截图 2026-05-04 181913.png" width="50%" height="50%" alt="图片描述" align="center" />

<img src="https://img.whitegoose.dpdns.org/image/屏幕截图 2026-05-04 182007.png" width="50%" height="50%" alt="图片描述" align="center" />

遍历一个顶点相连的所有超边，FM 算法在超图划分中的初始收益公式：$\text{gain}(i) = \text{FS}(i) - \text{TE}(i)$

- $FS(i)$：Fully on the Same Side（完全同边数），当且仅当顶点 $i$ 所在的分区中，只有自己在这个分区里，$FS(i) += 1$。

- $TE(i)$：Totally on the Edge（全同边数）当超边包含的所有顶点都在同一个分区时，$TE(i) += 1$。
- 如果有一条超边所有顶点都在左分区，那么 $TE(i)$ 会加 1，$gain(c)$ 会减 1，表示移动 $i$ 会让这条边变成割边，增加割代价。

以顶点 $c$ 为例，计算 $gain(c)$，与 $c$ 关联的超边有 3 条：

- e1: `a - c - e`
- e2: `b - c - d`
- e3: `c - f - e`

逐条超边计算 $FS(c) = 1$ 和 $TE(c) = 0$，$\text{gain}(c) = \text{FS}(c) - \text{TE}(c) = 1 - 0 = 1$

<table style="border:none;text-align:center;width:auto;margin: 0 auto;">
	<tbody>
		<tr>
			<!-- 修复：给 style 属性添加闭合引号 "，确保样式生效 -->
			<td style="padding: 6px"> <img src="https://img.whitegoose.dpdns.org/image/屏幕截图 2026-05-04 192635.png" style="width: 300px; height: auto;"> </td>
			<!-- 优化：给第二幅图也添加相同尺寸，避免两张图大小不一致 -->
			<td style="padding: 6px"> <img src="https://img.whitegoose.dpdns.org/image/屏幕截图 2026-05-04 192642.png" style="width: 300px; height: auto;"> </td>
		</tr>
        <tr>
		</tr>
	</tbody>
</table>

<table style="border:none;text-align:center;width:auto;margin: 0 auto;">
	<tbody>
		<tr>
			<!-- 修复：给 style 属性添加闭合引号 "，确保样式生效 -->
			<td style="padding: 6px"> <img src="https://img.whitegoose.dpdns.org/image/屏幕截图 2026-05-04 192741.png" style="width: 300px; height: auto;"> </td>
			<!-- 优化：给第二幅图也添加相同尺寸，避免两张图大小不一致 -->
			<td style="padding: 6px"> <img src="https://img.whitegoose.dpdns.org/image/屏幕截图 2026-05-04 192810.png" style="width: 300px; height: auto;"> </td>
		</tr>
        <tr>
		</tr>
	</tbody>
</table>

<table style="border:none;text-align:center;width:auto;margin: 0 auto;">
	<tbody>
		<tr>
			<!-- 修复：给 style 属性添加闭合引号 "，确保样式生效 -->
			<td style="padding: 6px"> <img src="https://img.whitegoose.dpdns.org/image/屏幕截图 2026-05-04 192821.png" style="width: 300px; height: auto;"> </td>
			<!-- 优化：给第二幅图也添加相同尺寸，避免两张图大小不一致 -->
			<td style="padding: 6px"> <img src="https://img.whitegoose.dpdns.org/image/屏幕截图 2026-05-04 192827.png" style="width: 300px; height: auto;"> </td>
		</tr>
        <tr>
		</tr>
	</tbody>
</table>

<table style="border:none;text-align:center;width:auto;margin: 0 auto;">
	<tbody>
		<tr>
			<!-- 修复：给 style 属性添加闭合引号 "，确保样式生效 -->
			<td style="padding: 6px"> <img src="https://img.whitegoose.dpdns.org/image/屏幕截图 2026-05-04 193035.png" style="width: 300px; height: auto;"> </td>
			<!-- 优化：给第二幅图也添加相同尺寸，避免两张图大小不一致 -->
			<td style="padding: 6px"> <img src="https://img.whitegoose.dpdns.org/image/屏幕截图 2026-05-04 193040.png" style="width: 300px; height: auto;"> </td>
		</tr>
        <tr>
		</tr>
	</tbody>
</table>

<table style="border:none;text-align:center;width:auto;margin: 0 auto;">
	<tbody>
		<tr>
			<!-- 修复：给 style 属性添加闭合引号 "，确保样式生效 -->
			<td style="padding: 6px"> <img src="https://img.whitegoose.dpdns.org/image/屏幕截图 2026-05-04 193052.png" style="width: 300px; height: auto;"> </td>
			<!-- 优化：给第二幅图也添加相同尺寸，避免两张图大小不一致 -->
			<td style="padding: 6px"> <img src="https://img.whitegoose.dpdns.org/image/屏幕截图 2026-05-04 193753.png" style="width: 350px; height: auto;"> </td>
		</tr>
        <tr>
		</tr>
	</tbody>
</table>

#### 复杂度分析

常数时间找最大收益节点：$O(1)$、更新节点收益：$O(1)$

一轮迭代里，每个节点只会被移动一次，每个邻居更新一次，整体复杂度是 $O(n)$。

#### 算法不足

- 过度依赖贪心策略
- 对划分初始状态十分敏感

以下图为例：在计算模块 A 和模块 B 当前的割代价收益时，收益完全一致。然而，移动 A 和移动 B 之后的割代价收益完全不同；

若先移动 B，则下一步移动与 B 相连的小模块可以获得较高的收益，因为所有模块都可以移动到一个划分内，显著减少割代价；

相反，若先移动 A，则移动与 A 相连的小模块收益一般，下一步的移动中不能收获较高的收益，始终存在相连的模块被分割的情况。

<img src="https://img.whitegoose.dpdns.org/image/屏幕截图 2026-05-04 195843.png" width="50%" height="50%" alt="图片描述" align="center" />



---

### 多层次划分算法



---

---

## 搜索算法

### 模拟退火



## 分析算法

### 谱聚类

### 基于网络流算法

