# [44] Çalli et al. 2015 · YCB 物体与模型集

> **原文**: B. Çalli, A. Walsman, A. Singh, S. S. Srinivasa, P. Abbeel, A. M. Dollar. *Benchmarking in manipulation research: The YCB object and model set and benchmarking protocols.* CoRR abs/1502.03143, 2015.
> **译名**: 《操作研究的基准测试：YCB 物体与模型集及基准协议》 — arXiv 预印本, 2015.

`#文献-数据集` `#里程碑`

---

## 1 · 研究问题

机器人抓取研究长期缺乏统一基准 — 每个团队拿不同物体做实验, 结果无法跨实验室对比。Yale + CMU + UC Berkeley (Çalli, Aaron Dollar, Pieter Abbeel 等) 联合的目标:

> 建立 **Yale-CMU-Berkeley (YCB) 物体集 + 标准评测协议**, 让全球抓取研究可比较 — 类似计算机视觉的 ImageNet 之于图像识别。

---

## 2 · 数据集组成

### 2.1 物理物体 (Physical Objects)

77 件日常物体, 5 大类:

| 类别 | 例子 | 数量 |
|---|---|---|
| **Food items** | 饼干盒、罐头、香蕉 | 16 |
| **Kitchen items** | 锅、碗、餐具 | 18 |
| **Tool items** | 锤子、扳手、剪刀 | 15 |
| **Shape items** | 球、立方体、圆环 | 17 |
| **Task items** | 积木、九柱戏 | 11 |

物体来自常见超市 / 五金店, 可购置复制。

### 2.2 数字模型 (3D Models)

每件物体提供:
- **高分辨率 mesh** (~$10^4-10^6$ 三角形)。
- **纹理图** (PBR 材质, 4096 × 4096)。
- **多视角 RGB-D 数据** (24-48 视角)。
- **物理参数** (质量, 重心, 摩擦, 弹性模量估计)。
- **不同分辨率版本** (高/中/低适合不同应用)。

### 2.3 数据格式

```json
{
  "object_id": "002_master_chef_can",
  "name": "Master Chef Coffee Can",
  "category": "food",
  "mass_kg": 0.414,
  "dimensions_mm": [102, 102, 139],
  "mesh_files": {
    "high_res": "google_512k/nontextured.ply",
    "low_res": "google_16k/nontextured.ply"
  },
  "textures": "google_512k/texture_map.png",
  "physical_props": {
    "static_friction": 0.4,
    "young_modulus_pa": 2e9,
    "poisson_ratio": 0.3
  }
}
```

---

## 3 · 基准协议 (核心贡献)

文章定义 6 套抓取/操作协议:

### 3.1 Pick & Place
- 起始: 物体在桌面随机姿态。
- 目标: 抓起并放到指定位置。
- 评分: 二值成功率 + 时间。

### 3.2 Stacking
- 多个物体堆叠。
- 评分: 堆叠高度。

### 3.3 Tool Use
- 用工具完成任务 (例如锤钉)。

### 3.4 Pouring
- 倒液体 / 颗粒。
- 评分: 倒出比例。

### 3.5 Threading
- 线穿针孔。
- 评分: 是否穿过。

### 3.6 Insertion
- 物体插入孔 (peg-in-hole)。
- 评分: 插入深度 + 力。

每协议规定起始 / 终止 / 评分标准, 引用 YCB 的论文必须 **报告物体 ID + 协议**。

---

## 4 · 数据采集设备

YCB 的高质量来自:
- **Carnegie Robotics 扫描仪**: 多视角激光扫描 + 高分 RGB。
- 自研三脚架转盘 (24 个视角自动旋转)。
- 透明背景便于分割。
- 物理参数: 天平 + 倾斜板 + 力学测试机。

整个采集流程 4-8 小时 / 物体。

---

## 5 · 关键贡献

1. **统一基准**: 全球抓取研究有了共同语言。
2. **多模态数据**: mesh + RGB-D + 物理参数。
3. **物理可复制**: 物体可在超市购买, 任何实验室能配同样数据集。
4. **评测协议**: 让结果跨论文可比。

---

## 6 · 局限

1. **物体选择偏向欧美**: 风格偏美式日用品, 与亚洲 / 欧洲 (KIT, [[Ref-48-Kasper-2012-KIT]]) 互补。
2. **形状有限**: 真实物体不能涵盖所有形状空间 (后由 EGAD, [[Ref-49-Morrison-2020-EGAD]] 用进化生成弥补)。
3. **mesh 质量参差**: 部分物体扫描有孔洞需修补。

---

## 7 · 历史影响

- 引用数 >2000, 抓取 / 操作研究的事实标准。
- 大多数后续抓取算法 (DexNet, AnyGrasp, [[Ref-46-Fang-2023-AnyGrasp]]) 都用 YCB 测试。
- 推动 **可复现性研究** 在机器人学。

---

## 8 · 与本论文 (Yi'25) 的关系

YCB 是本论文 **训练 + 测试数据基础**:
- In-domain: 45 件 YCB 训练神经代理。
- Out-of-domain: 28 件 (含 YCB 子集 + KIT + EGAD) 测试泛化。

没有 YCB, 本论文实验部分无法构建。

---

*← [[_参考文献索引]]*
