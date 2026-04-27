# [17] Kodnongbua et al. 2023 · 被动夹爪计算设计

> **原文**: M. Kodnongbua, I. G. Y. Lou, J. Lipton, A. Schulz. *Computational design of passive grippers.* arXiv:2306.03174, 2023.
> **译名**: 《被动夹爪的计算设计》 — arXiv 预印本, 2023.

`#文献-方法` `#规划式设计` `#被动夹爪`

---

## 1 · 研究问题

工业自动化中常见场景: **生产线上反复抓取同一种零件** (例如手机壳、瓶盖、连接器)。每次都用通用机械手抓既贵又慢, 但定制专用夹爪靠人工 CAD 又劳动密集。华盛顿大学 Adriana Schulz 团队的目标:

> **完全自动化** 给定 3D 物体 → 生成可 3D 打印的 **被动夹爪几何**, 使物体能 stably **坐入** 夹爪, 仅靠 **形状闭合 (form closure)** 抓取。**零电机, 零控制**。

应用场景: 装配线、医疗装载、产品包装、教学机器人。

---

## 2 · 形式化

### 2.1 输入

- 物体 3D mesh $O$。
- 期望抓取姿态 $T_\text{grasp} \in SE(3)$ (物体相对夹爪的 6D 位姿)。

### 2.2 设计变量

夹爪几何 $G$ 由 **支撑柱 (support pins / fingers)** 集合表示:
$$G = \{(\mathbf{p}_i, \mathbf{n}_i, h_i)\}_{i=1}^N$$
- $\mathbf{p}_i$: 第 $i$ 个支撑点位置。
- $\mathbf{n}_i$: 法线方向 (柱朝向)。
- $h_i$: 柱高度。

### 2.3 Form Closure 严格定义

物体被几何完全包围, 即使无摩擦也无法逃逸 — 所有 6 个自由度都被约束。

数学条件 (基于接触法线):
$$\text{positive span}\Big\{[\mathbf{n}_i;\; \mathbf{p}_i \times \mathbf{n}_i]\Big\}_{i=1}^N \;=\; \mathbb{R}^6$$

这是所谓 **wrench space** 张成 6D 全空间。

**最小接触数**: form closure 在 3D 物体上至少需 **7 个接触点** (positive span 定理)。

---

## 3 · 算法工作流

### 3.1 Step 1: 接触配置分析

对给定物体姿态, 分析表面区域:
- **支撑面** (法线朝上 +z): 可承重。
- **侧面** (法线水平): 可侧向卡。
- **倒勾面** (法线朝下 -z): 防物体逃逸。

每个面的有效性:
$$\text{score}(\mathbf{n}_i) = \cos(\angle(\mathbf{n}_i, \text{反重力}))$$

### 3.2 Step 2: 候选支撑点采样

从物体表面均匀采样若干 candidate $(\mathbf{p}_k, \mathbf{n}_k)$:
- 法线方向必须能让物体被包围。
- 排除柔软或薄弱区域。
- 共 $M \sim 10^3$ 候选。

### 3.3 Step 3: Form Closure 选择 (混合整数规划)

选择最少的支撑点子集 $S \subseteq \{1, ..., M\}$ 使得 form closure 成立:
$$\min_{S \subseteq \{1,...,M\}} \;\; |S|$$
$$\text{s.t.}\;\; \mathbf{0} \in \text{relative interior of conv}\Big\{[\mathbf{n}_i;\; \mathbf{p}_i \times \mathbf{n}_i]: i \in S\Big\}$$

用 SAT solver / MILP 求解。

### 3.4 Step 4: 可插入性 (Insertability)

物体必须能 **从某方向放入** 夹爪 (扫掠路径无碰撞):
$$\text{sweep}(O, T_0 \to T_\text{grasp}) \cap G = \emptyset$$

计算物体在某入射方向下的 swept volume, 检验与夹爪是否相交。若不能插入, 移除某些支撑点重试。

### 3.5 Step 5: 几何后处理

- 把支撑点集合连接成一体几何。
- 平滑 + 加固 (添加底座、连接梁)。
- 输出 STL, 直接 3D 打印。

---

## 4 · SDF 计算工具

为高效检验形状闭合 / 插入性, 预计算物体 SDF:
$$d(\mathbf{x}; O) = \begin{cases}
+\min_{\mathbf{y} \in \partial O} \|\mathbf{x} - \mathbf{y}\|, & \mathbf{x} \notin O \\
-\min_{\mathbf{y} \in \partial O} \|\mathbf{x} - \mathbf{y}\|, & \mathbf{x} \in O
\end{cases}$$

用 Marching Cubes / KD-Tree 加速。

---

## 5 · 实验

### 5.1 测试物体
- 100+ 件家居 / 工业物体。
- 形状: 规则、不规则、有倒勾、薄壳。
- 尺寸: 5 - 200 mm。

### 5.2 量化结果

| 指标 | 通用平行夹钳 | 本算法生成被动夹爪 |
|---|---|---|
| 抓取成功率 | ~70% | **~95%** |
| 设计时间 | 0 (现成) | ~分钟 (自动) |
| 制造成本 | 0 (现成) | ~$1-5/件 (3D 打印) |
| 可重定向 | ✅ | ❌ (固定姿态) |

### 5.3 制造与部署
- 输出 STL → FDM 打印 (PLA, PETG)。
- 装到机械臂末端或滑槽末端。
- 真机抓取验证 ~ 95% 成功率 (轻微容差)。

---

## 6 · 关键贡献

1. **完全自动化**: 从 mesh 到可制造 STL, 无需人工 CAD。
2. **零驱动**: 用形状闭合代替主动夹持。
3. **数学严格**: 基于 form closure 理论保证抓取稳定性。
4. **极速部署**: 同种零件大批量场景下成本极低。

---

## 7 · 局限

1. **每物一爪**: 不通用, 换零件就重新设计。
2. **依赖物体姿态**: 若物体到达姿态不准, 失败。
3. **不能动态**: 不能根据物体调整。
4. **无柔顺性**: 对脆弱物体可能损坏。

---

## 8 · 历史影响

- 推动 "**计算式定制夹爪**" 子方向。
- 与 [[Ref-26-Ha-2021-Fit2Form|Fit2Form]] 同属"自动夹爪生成"。
- 与 [[Ref-08-Zhao-2020-RoboGrammar|RoboGrammar]] 一道证明 **规划方法在机器人设计的有效性**。

---

## 9 · 与本论文 (Yi'25) 的关系

哲学对照:
- Kodnongbua: 极致定制 + 零驱动。
- 本论文: 通用平衡 + 单驱动 + 软体顺应。

两者都把"智能" 卸载到硬件, 但程度不同。

---

*← [[_参考文献索引]]*
