# [16] Feshbach et al. 2024 · CSC-Dubins 规划式机构树设计

> **原文**: D. Feshbach, W.-H. Chen, L. Xu, E. Schaumburg, I. Huang, C. Sung. *Algorithmic design of kinematic trees based on CSC Dubins planning for link shapes.* **WAFR 2024**.
> **译名**: 《基于 CSC Dubins 路径规划的运动学树连杆形状算法设计》 — 国际机器人算法基础研讨会 (WAFR), 2024.

`#文献-方法` `#规划式设计`

---

## 1 · 研究问题

机器人连杆通常是 **直线段**——但弯曲连杆可以更省空间、贴合表面、绕过障碍。如何系统化设计弯曲连杆形状? UPenn Cynthia Sung 团队的目标:

> 借用 **Dubins 路径** (经典最短路径) 把"机器人连杆形状设计" 转化为 **规划问题**: 给定起点位姿、终点位姿、最小转弯半径, 规划一条合法的弯曲路径作为连杆。

---

## 2 · Dubins 路径基础

### 2.1 Dubins 1957 定理

在 **平面 + 转弯半径下界 $\rho_\min$** 约束下, 从配置 $\mathbf{q}_0 = (x_0, y_0, \theta_0)$ 到 $\mathbf{q}_1 = (x_1, y_1, \theta_1)$ 的 **最短路径** 必属于 6 种之一:
$$\{\text{LSL}, \text{LSR}, \text{RSL}, \text{RSR}, \text{LRL}, \text{RLR}\}$$
- L = 左转弧 (radius $\rho$)。
- R = 右转弧。
- S = 直线段。

最常用的是 **CSC** 类型 (弧 + 直 + 弧)。

### 2.2 路径方程

#### 单段弧 (左转, 半径 $\rho$, 圆心 $\mathbf{c} = (x_c, y_c)$)
$$x(s) = x_0 + \rho\sin(\theta_0 + s/\rho) - \rho\sin\theta_0$$
$$y(s) = y_0 - \rho\cos(\theta_0 + s/\rho) + \rho\cos\theta_0$$
$$\theta(s) = \theta_0 + s/\rho$$
$s$: 弧长 $\in [0, \rho \alpha]$, $\alpha$: 弧角。

#### 单段直线
$$x(s) = x_0 + s\cos\theta_0,\;\; y(s) = y_0 + s\sin\theta_0,\;\; \theta(s) = \theta_0$$

### 2.3 CSC 总长度
$$L_\text{CSC} = \rho|\alpha_1| + d + \rho|\alpha_2|$$
- $\alpha_1, \alpha_2$: 两端弧角。
- $d$: 中间直线长度。

可解析求出 6 种类型每种的长度, 选最短为 Dubins 距离。

---

## 3 · 应用到连杆设计

### 3.1 重新解读

把 **机器人连杆** 视为 Dubins 路径:
- 起点 = 关节 1 处的位姿 $(p_1, \tau_1)$ (位置 + 切向)。
- 终点 = 关节 2 处的位姿 $(p_2, \tau_2)$。
- 路径 = 连杆形状 = CSC 曲线。

设计变量: $(\rho_\min, \alpha_1, d, \alpha_2)$。

### 3.2 Kinematic Tree 扩展

对多关节机器人 (kinematic tree):
- 树根是基座。
- 每个 link 是 CSC 曲线。
- 关节是 CSC 切点。
- 切线方向连续 (类似 $G^1$ 几何)。

数学约束:
- 切线连续: $\tau_\text{end of link i} = \tau_\text{start of link i+1}$。
- 关节空间限制: 每关节有最大角度 $\theta_\text{max}$。
- 自碰撞约束: 连杆间不相撞。

### 3.3 末端位姿可达性问题

给定一组目标末端位姿 $\{q_\text{ee}^{(k)}\}$, 求树形机器人结构使得每个 $q_\text{ee}^{(k)}$ 可达:
$$\min_{\text{树形结构}} \;\; \sum_k \text{cost}(q_\text{ee}^{(k)} \to q_\text{achieved}) \;+\; \lambda \cdot \text{material}$$

---

## 4 · 算法

```
─── Algorithm: Dubins Tree Design ──────────
输入: 树拓扑 (n 关节), 末端位姿目标集 {q_ee^{(k)}}
输出: 每条连杆的 CSC 参数

1) 关节位置初始化 (随机 / 启发式)
2) 重复直至收敛:
    a) 对每条连杆 (从根到叶):
        给定起点终点, 解 6 种 Dubins 类型, 选最短
        记录 (ρ, α_1, d, α_2)
    b) 评估末端位姿误差
    c) 若误差 > 阈值:
        微调关节位置 (梯度下降 / CMA-ES)
3) 验证: 自碰撞、可制造、关节角约束
4) 输出 STL → 激光切割 / 折叠
```

---

## 5 · 制造路径

### 5.1 Planar (平面)
- 用 **激光切割** 平板 (亚克力, PET, 薄铝)。
- CSC 曲线直接画在平板上。
- 关节用销 + 弹簧 / 柔性铰链。

### 5.2 Foldable (Origami)
- 用 **折叠** 把平面 CSC 起步成 3D。
- 参考 Miura 折等数学折叠模式。

---

## 6 · 实验

### 6.1 演示场景
- 2 自由度机器人在工作空间中绕过障碍。
- 4 自由度树形手臂可达多个目标位姿。

### 6.2 量化结果

| 设计 | 弯曲连杆 | 直线连杆 |
|---|---|---|
| 绕过障碍 | 80% 成功 | 30% 成功 |
| 末端位姿误差 | < 5 mm | < 5 mm |
| 材料用量 | 1.0× | 0.85× (略省) |

弯曲连杆牺牲少量材料换取大幅工作空间提升。

### 6.3 制造验证
- 激光切割亚克力 + 销关节。
- 实测末端位姿与算法预测一致 (~2 mm 误差)。

---

## 7 · 关键贡献

1. **Dubins → 连杆形状**: 把经典运动规划工具迁移到机器人设计。
2. **数学严格**: 6 种类型解析求最短, 无需数值优化连杆形状。
3. **可制造**: 平面 + 折叠输出实物。
4. **树形扩展**: 多自由度机器人统一框架。

---

## 8 · 局限

1. **平面假设**: 真 3D 设计需 3D Dubins (非平凡)。
2. **不优化拓扑**: 树结构是预设的。
3. **不考虑动力学**: 仅运动学。
4. **关节假设固定类型**: 旋转副为主。

---

## 9 · 历史影响

- WAFR (Workshop on the Algorithmic Foundations of Robotics) 经典议题。
- 推动 **形式化机器人设计** 子方向。
- 启发后续 Origami 机器人 + 微型可折叠器件。

---

## 10 · 与本论文 (Yi'25) 的关系

互补关系: Feshbach 处理 **几何形态** (规划式), 本论文处理 **材料分布** (优化式)。两者都是 co-design 子问题, 未来可结合 (用 Dubins 选拓扑骨架 + NN 代理优化局部刚度)。

---

*← [[_参考文献索引]]*
