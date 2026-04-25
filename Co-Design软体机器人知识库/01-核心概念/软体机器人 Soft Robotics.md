# 软体机器人 Soft Robotics

> **一句话理解**：用 **柔顺材料** 制作的机器人，能像人手一样变形贴合物体，而不是靠刚性关节拼接。

`#核心概念`

---

## 1. 什么是软体机器人？

**软体机器人 (Soft Robot)** 指机体主要由柔顺材料 (TPU、硅胶、水凝胶等) 构成的机器人。它们没有传统意义上的"铰链 + 轴承 + 马达"结构，而是把运动功能 **编码到身体材料的几何里**。

Rus & Tolley (2015) 在 Nature 发表的经典综述 ([[../03-参考文献/Ref-01-Rus-2015-软体机器人综述]]) 定义了这个领域的基本面貌：**设计-制造-控制** 三个方面都与传统机器人不同。

> 📌 灵感来源：章鱼触手、大象鼻子、毛毛虫身体、人类舌头——这些"无骨骼或少骨骼"的生物器官能做出极其复杂的变形。

---

## 2. 软 vs 硬 的优劣对比

| 维度 | 软体机器人 | 刚性机器人 |
|---|---|---|
| 安全性 | 🟢 与人共处安全 | 🔴 关节碰撞危险 |
| 承载力 | 🔴 易变形失稳 | 🟢 可承大负载 |
| 几何适应性 | 🟢 任意形状贴合 | 🔴 需形状匹配夹具 |
| 控制精度 | 🔴 非线性迟滞 | 🟢 可达毫米级 |
| 建模难度 | 🔴 无穷自由度 | 🟢 多刚体动力学 |
| 制造 | 🟢 可一体成型 | 🔴 多零件装配 |

正是因为"适应性"与"承载力"的矛盾，催生了 **Co-Design**：让机器人自己学出最适合任务的软硬度分布，见 [[Co-Design 协同设计]]。

---

## 3. 软体机器人的关键技术栈

### 3.1 执行器 (Actuator)

- **气动 (pneumatic)**：充气膨胀——章鱼抓手常用。见 [[../03-参考文献/Ref-36-Zhai-2023-DesktopFabrication]]。
- **腱驱动 (tendon-driven)**：拉线拉扯变形——本论文方案。详见 [[腱驱动 Tendon-driven]]。
- **电活性聚合物 (EAP)**：通电形变。
- **磁驱动/液压/形状记忆合金**：新兴方向。

### 3.2 制造工艺

- **模塑铸造**：倒模硅胶，见 [[../03-参考文献/Ref-32-Bilodeau-2015-MonolithicSensors|Bilodeau'15]]、[[../03-参考文献/Ref-33-Rothemund-2018-BistableValve|Rothemund'18]]。
- **3D 打印**：TPU/硅胶/水凝胶打印，见 [[../03-参考文献/Ref-35-Hubbard-2021-3DPrintedFluidic]]、[[../03-参考文献/Ref-36-Zhai-2023-DesktopFabrication]]。
- **变刚度复合**：层间摩擦、相变、颗粒阻塞——见 [[../03-参考文献/Ref-37-Kim-2019-VariableStiffness]]、[[../03-参考文献/Ref-38-Narang-2017-LaminarJamming]]、[[../03-参考文献/Ref-34-Wei-2016-ParticleJamming]]。

### 3.3 建模与模拟

- **FEM** 有限元：精确但慢。详见 [[FEM 有限元法]]。
- **简化模型**：如常曲率假设 (PCC)。
- **可微分模拟**：[[../03-参考文献/Ref-18-Macklin-2022-Warp|Warp]]、[[../03-参考文献/Ref-19-Hu-2020-DiffTaichi|DiffTaichi]]、[[../03-参考文献/Ref-41-Faure-2012-SOFA|SOFA]]。
- **神经物理代理**：本论文方案。见 [[神经物理模拟器 Neural Physics]]。

### 3.4 控制

- 传统 PID/Feedforward
- 基于学习：强化学习、模仿学习、残差物理 ([[../03-参考文献/Ref-11-Gao-2024-ResidualPhysics]])
- 基于位姿采样 + 代理评估 (本论文)

---

## 4. 软体夹爪 (Soft Gripper)

软体夹爪是最成功的软体机器人子方向，Shintake 等在 Advanced Materials 上综述了所有主流方案 ([[../03-参考文献/Ref-02-Shintake-2018-软体夹爪综述]])。

四类机制：
1. **Actuation (主动驱动型)**：气动、腱驱动 (本论文)。
2. **Controlled stiffness (变刚度型)**：颗粒阻塞、层间阻塞。
3. **Controlled adhesion (可控粘附型)**：壁虎抓附、静电、磁性。
4. **Underactuated (欠驱动型)**：被动适应形状，如 [[../03-参考文献/Ref-10-Odhner-2014-SDMhand|SDM Hand]]。

---

## 5. 本论文涉及的软体设计要素

| 要素 | 设计变量 | 参考 |
|---|---|---|
| 材料 | NinjaFlex TPU 95A | 硬件章节 |
| 结构 | flexure joint (薄梁) | [[柔性铰链 Flexure Joint]], [[../03-参考文献/Ref-39-Howell-2013-CompliantMechanisms]] |
| 驱动 | 腱 + 单电机 | [[腱驱动 Tendon-driven]], [[../03-参考文献/Ref-42-Hirose-1978-UniformPressure]] |
| 刚度分布 | 22 段 block 各自独立 | [[刚度分布 Stiffness Distribution]] |
| 制造 | 3D 打印 (infill + wall loops) | 论文 §4.2 |

---

## 6. 延伸阅读

- Rus & Tolley 2015 综述：[[../03-参考文献/Ref-01-Rus-2015-软体机器人综述]]
- Shintake 2018 软夹爪综述：[[../03-参考文献/Ref-02-Shintake-2018-软体夹爪综述]]
- Piazza 2019 机器手百年史：[[../03-参考文献/Ref-31-Piazza-2019-机器手百年]]
- Chen & Wang 2020 软体设计综述：[[../03-参考文献/Ref-13-Chen-2020-软体设计综述]]

---

*← 返回 [[../00-总览/主索引 MOC]]*
