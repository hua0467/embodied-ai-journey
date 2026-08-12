<p align="center">
  <h1 align="center">embodied-ai-journey</h1>
  <p align="center"><strong>汽车工程跨域具身智能实习全记录</strong><br>用车辆控制思维拆解机器狗、VLA、LLM 工程落地</p>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/domain-Embodied_AI-blue" alt="Embodied AI">
  <img src="https://img.shields.io/badge/domain-VLA_Data_Pipeline-green" alt="VLA">
  <img src="https://img.shields.io/badge/domain-Robot_Control-orange" alt="Robot Control">
  <img src="https://img.shields.io/badge/status-internship_2026-lightgrey" alt="Status">
  <img src="https://img.shields.io/badge/license-MIT-yellow" alt="License">
  <img src="https://img.shields.io/badge/updates-weekly-brightgreen" alt="Updates">
</p>

---

## 快速了解

> **全网独一份「车辆控制 → 具身智能」完整映射工程笔记。** 不是教程合集——是一个汽车工程实习生从零跨界机器人的真实记录：SLAM、MPC、VLA 数据管线、RAG 企业落地、VR 遥操作全链路。每篇都附汽车-机器人对照表，让你用已有的控制思维理解 AI。

---

## 🎯 这个仓库解决什么问题

| 痛点 | 这里的解法 |
|------|-----------|
| 工科生转 AI 无从下手 | 用你已经会的控制理论做桥梁，不是从零学 AI |
| 具身智能教程全是 CS 背景写的 | 从车辆工程视角重新拆解，传感器→ECU→执行器 全对上 |
| 实习笔记太散、没有工程价值 | 每篇可独立阅读，附对照表 + 踩坑记录 + 可复用代码片段 |
| 企业 RAG/Agent 落地只有理论 | 工业巡检机器狗真实场景，从数据清洗到检索评测完整闭环 |

---

## 📂 目录速查

```
embodied-ai-journey/
├── README.md                              ← You are here / 仓库门面
├── LICENSE                                (MIT)
├── CHANGELOG.md                           (更新日志)
│
├── 00-getting-started/                    ← 新读者入口 · New? Start here
│   └── 4天从汽车到机器狗.md                 Auto→Embodied AI: A 4-Day Cross-Domain Diary
│
├── 01-internship-toolchain/               ← 工具链 · Toolchain
│   ├── GitHub从零发布完全指南.md             GitHub from Zero: Intern's First Open Source Guide
│   └── AI工具协同作战-企业级项目落地方法论.md   Multi-AI Collaboration: From Solo to Engineering Team
│
├── 02-vla-data-pipeline/                  ← VLA 数据管线 · Data Pipeline
│   ├── 从手到数据-VR遥操管线到底在跑什么.md     VR Teleop Pipeline: From Hand Tracking to Training Data
│   └── 给机器人基础模型准备教材-openpi数据格式学习笔记.md  π₀.₅ Data Format: LeRobot v3.0 & OpenPI Inside Out
│
├── 03-robot-dog-field-notes/              ← 规划中 · Planned
├── 04-vehicle-to-robot-control-mapping/   ← 规划中 · Planned
└── code/                                  ← 规划中：可复现脚本
```

---

## 🧭 三类读者，三条路径

<details open>
<summary><strong>🅰️ 工科跨 AI 实习生</strong></summary>

从 [4天从汽车到机器狗](00-getting-started/4天从汽车到机器狗.md) 开始 → 然后读 [AI工具协同作战](01-internship-toolchain/AI工具协同作战-企业级项目落地方法论.md) 学怎么用 AI 提高效率 → 需要搭 GitHub 时看 [GitHub 完全指南](01-internship-toolchain/GitHub从零发布完全指南.md)
</details>

<details>
<summary><strong>🅱️ 机器人 / 自动驾驶研发</strong></summary>

直奔 [VR遥操管线](02-vla-data-pipeline/从手到数据-VR遥操管线到底在跑什么.md) 和 [openpi 数据格式笔记](02-vla-data-pipeline/给机器人基础模型准备教材-openpi数据格式学习笔记.md) → 关注每篇文末的「汽车-机器人对照表」→ 等 [整车-机器人控制对照手册](04-vehicle-to-robot-control-mapping/) 更新
</details>

<details>
<summary><strong>🅲️ 校招求职</strong></summary>

看 [AI工具协同作战](01-internship-toolchain/AI工具协同作战-企业级项目落地方法论.md) 证明工程落地能力 → 看 [VR遥操管线](02-vla-data-pipeline/从手到数据-VR遥操管线到底在跑什么.md) 证明系统思维 → README 底部 Roadmap 证明持续产出能力 → 简历引用本仓库作为 PoW（工作量证明）
</details>

---

## 🛠 技术栈

```
具身智能：SLAM · MPC · A* · π₀.₅ · LeRobot v3.0 · OpenPI · Isaac Sim
车辆控制：CAN 总线 · PID · ECU MAP 标定 · IMU · 多传感器融合
LLM/RAG：LangChain · 向量检索 · 语义分块 · Agent 四种范式 · Prompt Engineering
工程工具：Git · GitHub Actions · FastAPI · WebSocket · Docker · Python
数据管线：UDP · HDF5 · Parquet · 分位数归一化 · 逆运动学(IK) · URDF
```

---

## 🚀 Roadmap

| 阶段 | 内容 | 状态 |
|------|------|------|
| **已完成** | 4天入门日记、GitHub 指南、AI 工具协同方法论 | ✅ |
| **已完成** | VR 遥操管线全链路、openpi 数据格式学习笔记 | ✅ |
| **进行中** | README 门面重构、文档规范化、中英文命名 | 🔄 |
| **短期** | 机器狗实操笔记（SLAM 建图调参、MPC 步态力控、A* 路径规划踩坑） | 📋 |
| **短期** | 企业级 RAG 落地（巡检知识库原型、分块策略对比、检索评测） | 📋 |
| **中期** | 整车-机器人控制对照手册（ECU↔LLM、ESP↔力控、CAN↔WebSocket） | 📋 |
| **中期** | 人形机器人数据管线（F1 VR 采集 → IK 解算 → 训练数据交付全流程） | 📋 |
| **长期** | 机器狗巡检 RAG 知识库 Demo + 脱敏数据集 | 💡 |
| **长期** | 《汽车工程师转具身智能学习路线图》完整版 | 💡 |

---

## 👤 作者

**keywashion（梓华）** — 汽车工程大三，具身智能研发助理实习生。

大学挂过高数、线代课本没怎么翻过。但有一个不谦虚的优点：**给我一个东西，我能从底层把它拆明白。** 三个月前连「大模型」和「大数据」都分不清，现在在搭 VLA 数据管线。

> 成绩单不反映动手能力。贪玩的人往往好奇心重，坐不住的人往往动手能力强。这个行业变得太快——真正稀缺的不是「什么都会的人」，是「能快速学会任何东西的人」。

- 📧 1873733846@qq.com
- 🐙 [github.com/hua0467](https://github.com/hua0467)

---

## 📄 其他

- **License**：[MIT](LICENSE) — 代码和文档都可以自由使用，署名即可
- **更新日志**：[CHANGELOG.md](CHANGELOG.md)
- **纠错/交流**：提 Issue 或直接发邮件。被纠错 = 白嫖了一节课，不亏。

---

<p align="center"><em>持续更新中 | 2026.08 | 一个非典型学生的非典型学习记录</em></p>
