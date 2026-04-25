# Sim-to-Real 仿真到现实迁移

> **一句话理解**：把在仿真里学到的东西 (策略、设计、刚度) 搬到真机器人上，让它 **在现实中也能工作**——是机器人学习中最棘手的问题之一。

`#核心概念`

---

## 1. 为什么 Sim-to-Real 难？

- 仿真器默认接触模型与真实不同 (摩擦、变形、反冲)。
- 材料属性测量误差 (杨氏模量、阻尼)。
- 传感噪声、延迟、电机非线性。
- 数据分布差异 (DR: Domain Randomization 部分缓解)。

这些在软体机器人上 **尤其严重**：材料行为随温度、打印批次、使用次数变化。

---

## 2. Sim-to-Real 的常见策略

| 策略 | 思路 | 代表工作 |
|---|---|---|
| **域随机化 (DR)** | 仿真中大范围随机扰动 | OpenAI Rubik's Cube |
| **系统辨识 (SI)** | 真机数据拟合仿真参数 | 经典 |
| **残差物理** | 学习仿真与真实的差值 | [[../03-参考文献/Ref-11-Gao-2024-ResidualPhysics]] |
| **域对齐** | CycleGAN 等把仿真图迁移为真实感 | — |
| **Real-in-the-Loop** | 真机反馈在线微调 | — |

---

## 3. 本论文的 Sim-to-Real 路径

本文把 Sim-to-Real 简化为 **"仿真刚度 → 真实打印参数"** 的标定问题：

```
仿真刚度 log E = {13.5, ..., 17.0}
        │
        ▼  (本论文 §6.3)
3D 打印结构参数  (wall loops, infill, ...)
        │
        ▼  (三点弯曲/压痕测试)
实测 log E
        │
        ▼  (线性映射)
匹配到打印参数
```

具体做法：
- 测试机：Mark-10 力计 + 万能测试机。
- 试样：每种打印参数组合 3 件，每件测 3 次。
- Flexure 块：三点弯曲 (论文 Fig 5b)。
- Segment 块：压痕测试 (论文 Fig 5c)。

### 材料统一、结构变化
- 全部用 NinjaFlex Cheetah (TPU 95A) —— 统一材料。
- 仅通过 wall loops / infill / infill direction 改变结构级刚度。

### 结果 (§4.3 真机实验)
- 10 个物体中优化夹爪绝大多数优于 rigid 和 soft 基线，尤其在 >100g 重物和不规则几何上。
- Mustard bottle / Pear 等典型物体仿真 vs 真机成功率接近 (Table 3b)。

---

## 4. 与其他 Co-Design 论文的 Sim-to-Real

- **Schaff 2023 Crawler** ([[../03-参考文献/Ref-06-Schaff-2023-CrawlerSimToReal]])：用 RL 在仿真共优化软爬行机器人 + 策略，再 DR 到真机。
- **He 2024 MORPH** ([[../03-参考文献/Ref-07-He-2024-MORPH]])：训练可微硬件代理，策略在真机微调。
- **Gao 2024** ([[../03-参考文献/Ref-11-Gao-2024-ResidualPhysics]])：残差物理补偿 sim-real gap，软体臂直接部署。

这些工作共同启示：**co-design 的成败极大取决于 Sim-to-Real 质量**——本论文通过 (a) 限制设计空间为材料参数，(b) 精确标定，(c) 统一材料，极大简化了 Sim-to-Real。

---

## 5. 关联概念

- [[Co-Design 协同设计]]
- [[刚度分布 Stiffness Distribution]]
- [[神经物理模拟器 Neural Physics]]
- [[../02-方法分类/梯度优化方法]]
- [[../03-参考文献/Ref-06-Schaff-2023-CrawlerSimToReal]]
- [[../03-参考文献/Ref-11-Gao-2024-ResidualPhysics]]

---

*← 返回 [[../00-总览/主索引 MOC]]*
