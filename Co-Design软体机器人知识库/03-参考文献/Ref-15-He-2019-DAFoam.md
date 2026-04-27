# [15] He et al. 2019 · DAFoam: 开源伴随法 MDO 框架

> **原文**: P. He, C. Mader, J. Martins, K. Maki. *DAFoam: An open-source adjoint framework for multidisciplinary design optimization with OpenFOAM.* **AIAA Journal**, 2019. doi:10.2514/1.J058853
> **译名**: 《DAFoam: 基于 OpenFOAM 的多学科设计优化开源伴随框架》 — *AIAA 学报*, 2019.

`#文献-工具` `#CFD` `#伴随法`

---

## 1 · 工具定位

OpenFOAM 是工业级开源 CFD 求解器, 但 **本身不带伴随法**。Michigan Joaquim Martins 团队的目标:

> 在 OpenFOAM 之上构建 **可微分 + 伴随法** 工具集 **DAFoam**, 让大规模 ($\sim 10^6$ 设计变量) 多学科优化对开源用户可达。

---

## 2 · 离散伴随法 vs 连续伴随法

### 2.1 离散伴随法 (DAFoam 选)

先离散化 PDE 得代数方程, 再构建伴随:
$$\mathbf{R}(\mathbf{u}, \mathbf{x}) = 0\;\;\Rightarrow\;\;\left(\frac{\partial \mathbf{R}}{\partial \mathbf{u}}\right)^\top \boldsymbol{\lambda} = -\frac{\partial J}{\partial \mathbf{u}}^\top$$

**变量含义**：
- $\mathbf{R}$ — 离散方程残差。
- $\mathbf{u}$ — 离散流场状态。
- $\mathbf{x}$ — 设计变量 (几何控制点)。
- $J$ — 目标函数 (例如阻力)。
- $\boldsymbol{\lambda}$ — 伴随变量 (拉格朗日乘子)。
- $\frac{\partial \mathbf{R}}{\partial \mathbf{u}}$ — CFD 雅可比矩阵 (大稀疏矩阵)。

**公式解读**：详见 [[Ref-14-Buckley-2010-Airfoil]] 完整推导。**离散伴随的优势**:
- 与求解器一致 (无离散化误差)。
- 自动微分 (AD) 可生成。
- 工业界常用。

### 2.2 实现

DAFoam 用 **Tapenade / CoDiPack** 等 AD 工具自动产生 OpenFOAM 求解器的伴随版本:

```
原 OpenFOAM: solve(R, U)
   ↓ AD 生成
DAFoam:     solveAdjoint(R, U, λ, J)
```

---

## 3 · 多学科耦合 (MDO)

### 3.1 联合优化目标

$$\min_\mathbf{x} \;\; \sum_{k=1}^{K} w_k J_k(\mathbf{u}^{(k)}, \mathbf{x})$$

**变量含义**：
- $k$ — 学科索引 (空气动力, 结构, 热, 声学)。
- $K$ — 学科总数。
- $J_k$ — 第 $k$ 学科目标函数。
- $\mathbf{u}^{(k)}$ — 学科 $k$ 状态变量。
- $\mathbf{x}$ — 共享设计变量 (几何)。
- $w_k$ — 学科权重。

**公式解读**：多学科目标加权和, 共享同一组几何设计变量。

### 3.2 耦合伴随

学科间可能耦合 (例如空气动力影响翼面变形, 变形又改气动)。DAFoam 用 **Block Gauss-Seidel** 迭代求解耦合伴随:

$$\boldsymbol{\lambda}^{(k+1)} = \text{solve}\left[\frac{\partial \mathbf{R}^{(k)}}{\partial \mathbf{u}^{(k)}}^\top, -\frac{\partial J}{\partial \mathbf{u}^{(k)}}^\top - \sum_{j \neq k} \frac{\partial \mathbf{R}^{(j)}}{\partial \mathbf{u}^{(k)}}^\top \boldsymbol{\lambda}^{(j)}\right]$$

**变量含义**：
- $\boldsymbol{\lambda}^{(k)}$ — 第 $k$ 学科的伴随变量。
- $\frac{\partial \mathbf{R}^{(j)}}{\partial \mathbf{u}^{(k)}}$ — 学科 $j$ 残差对学科 $k$ 状态的导数 (耦合项)。

**公式解读**：每个学科伴随方程要考虑 **其他学科** 通过状态变量的间接影响。Block Gauss-Seidel 迭代轮流求解每个学科, 直到收敛。

---

## 4 · 与 OpenMDAO 集成

NASA 的 **OpenMDAO** 是多学科优化框架, DAFoam 通过 component API 集成:

```python
from openmdao.api import Problem, Group
from dafoam import DAFoam

prob = Problem()
prob.model.add_subsystem('aero', DAFoam(...))
prob.model.add_subsystem('struct', StructSolver(...))
prob.model.add_design_var('x_design')
prob.model.add_objective('drag_total')
prob.model.add_constraint('lift', lower=L_min)

prob.driver = pyOptSparseDriver()
prob.driver.options['optimizer'] = 'SNOPT'
prob.run_driver()
```

设计师只需声明 (设计变量, 目标, 约束), DAFoam 自动算梯度并交给优化器 (SNOPT, IPOPT, SLSQP)。

---

## 5 · 工业应用

DAFoam 已被用于:
- **波音、空客** 翼型优化。
- **汽车制造商** 外形减阻 (10-30% 减阻)。
- **风机叶片** 设计 (功率系数 $C_p$ 提升)。
- **涡轮叶片** 多学科 (气动 + 热 + 应力)。

设计变量规模: $\sim 10^4 - 10^6$ 控制点。

---

## 6 · 性能基准

DAFoam 论文报告:
- 翼型 ($n \sim 100$): 单次梯度计算 ~5 min, 100 代收敛 ~10 hr。
- NACA 0012 减阻: 阻力 -25%, 形状变化合理。
- 与商业 ANSYS Adjoint 对比: 速度相当, 精度更好 (开源代码可调)。

---

## 7 · 关键贡献

1. **开源伴随 OpenFOAM**: 让大规模 CFD 优化对学界可达。
2. **多学科耦合**: 气动 + 结构 + 热联合优化。
3. **与 OpenMDAO 集成**: 完整 MDO 工业管线。
4. **可扩展**: 用户可加自己学科的求解器。

---

## 8 · 局限

1. **依赖 OpenFOAM 代码**: OpenFOAM 升级时 DAFoam 也要跟。
2. **AD 内存爆**: 大网格时 AD tape 内存极大, 需 checkpointing。
3. **接触 / 大变形受限**: OpenFOAM 主要做流体, 软体力学需扩展。

---

## 9 · 历史影响

- 推动开源 MDO 工具生态。
- 与商业 ANSYS Adjoint / Star-CCM+ 形成竞争。
- 被超过 1000 个研究小组使用。

---

## 10 · 与本论文 (Yi'25) 的关系

作为 **工程界可微分 MDO 已成熟** 的论据。本论文软体接触场景下 OpenFOAM + 伴随法不直接适用, 转用 NN 代理替代伴随。

---

*← [[_参考文献索引]]*
