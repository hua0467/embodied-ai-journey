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

> **全网独一份「车辆控制 → 具身智能」完整映射工程笔记 + 一个人的成长轨迹。** 不是教程合集——是一个汽车工程实习生从零跨界机器人的真实记录：SLAM、MPC、VLA 数据管线、RAG 企业落地、VR/真机遥操作全链路。每篇都附汽车-机器人对照表，让你用已有的控制思维理解 AI。同时，这个仓库也是一个普通实习生从"只会修 bug"逐步训练商业嗅觉的**进行时**记录——早期笔记保留当时的认知局限，后期思考单独立目录、标注瑕疵、持续迭代。

---

## 📈 人物与仓库完整成长轨迹（进行时，仍在迭代，存在认知瑕疵）

| 阶段 | 对应目录 | 说明 |
|------|----------|------|
| **阶段① 短期实习阶段** | `00-getting-started` `01-internship-toolchain` `02-vla-data-pipeline` | 刚接触机器人项目十几天，聚焦解决工程 bug、吃透 IK、遥操作数据管线模块。此时只看见技术实现，对商业、客户、供应链几乎没有概念，存在不少认知盲区。**文档原样保留，不作美化修改。** |
| **阶段② 产业观察与复盘（8.26 ARE 展会起）** | `03-post_intern_iterate` `04-vertical_case_study` `05-mvp_practice` | 2026.8.26 ARE 展会（第18届深圳国际工业自动化及机器人展览会）之后，在已有的工程经验基础上，向外拓展产业、商业、场景、创业维度思考。**这一部分是持续迭代的，会不断推翻、修正自己过去的观点，存在错误与片面判断，属于个人成长记录，不是成熟行业报告。** 目标：训练自己的产业嗅觉，向硬科技产业新贵的方向持续成长。 |

> 🚧 当前进度：阶段②目录已于 2026.8.13 建立骨架（目录与提纲），正文自 2026.8.26 ARE 2026 展会起逐步填充——**这是规划，不是已完成的东西，如实展示。**

> ⚠️ **免责声明**：所有后期商业与产业分析仅为本本科生个人学习推演，没有量产产品与创业实战经历，内容会迭代修正，**不构成任何投资、创业决策建议**。

---

## 🎯 这个仓库解决什么问题

| 痛点 | 这里的解法 |
|------|-----------|
| 工科生转 AI 无从下手 | 用你已经会的控制理论做桥梁，不是从零学 AI |
| 具身智能教程全是 CS 背景写的 | 从车辆工程视角重新拆解，传感器→ECU→执行器 全对上 |
| 实习笔记太散、没有工程价值 | 每篇可独立阅读，附对照表 + 踩坑记录 + 可复用代码片段 |
| 企业 RAG/Agent 落地只有理论 | 工业巡检机器狗真实场景，从数据清洗到检索评测完整闭环 |
| 只有技术视角，看不懂产业 | 阶段②目录：赛道拆解、伪需求过滤、风险清单、场景案例，标注瑕疵持续迭代 |

---

## 📂 目录速查

```
embodied-ai-journey/
├── README.md                              ← You are here / 仓库门面
├── LICENSE                                (MIT)
├── CHANGELOG.md                           (更新日志)
│
├── 00-getting-started/                    ← 实习原始记录 · New? Start here（冻结区，只增不改）
│   └── 4天从汽车到机器狗.md                 Auto→Embodied AI: A 4-Day Cross-Domain Diary
│
├── 01-internship-toolchain/               ← 实习原始记录 · Toolchain（冻结区）
│   ├── GitHub从零发布完全指南.md             GitHub from Zero: Intern's First Open Source Guide
│   └── AI工具协同作战-企业级项目落地方法论.md   Multi-AI Collaboration: From Solo to Engineering Team
│
├── 02-vla-data-pipeline/                  ← 实习原始记录 · Data Pipeline（冻结区）
│   ├── _02_post_ref_index.md              ← 后期复盘索引（新增，只做外部链接，不动旧文）
│   ├── 从手到数据-VR遥操管线到底在跑什么.md     VR Teleop Pipeline: From Hand Tracking to Training Data
│   ├── 给机器人基础模型准备教材-openpi数据格式学习笔记.md  π₀.₅ Data Format: LeRobot v3.0 & OpenPI Inside Out
│   ├── IK求解器从v1到v2重构实录.md             IK Solver v2: 6 Bugs Fixed, Global 17-DOF, Unit Tests
│   ├── 真机遥操第一天-机器人动了也拧了.md        Real-Robot Teleop Day 1: It Moved, and It Twisted
│   └── 真机遥操第二天-夹爪信号去哪了.md          Real-Robot Teleop Day 2: The Missing Gripper Signal
│
├── 03-post_intern_iterate/                ← ✨后期迭代思考 · Growth Line（骨架，正文迭代中）
│   ├── 00_my_growth_journey.md            成长自述：从修 bug 到看生意
│   ├── 01_industry_track_decompose.md     赛道拆解：Demo 场景与付费场景
│   ├── 02_industry_myth_and_fake_demand.md 行业幻觉与伪需求
│   ├── 03_data_infra_border_and_barrier.md 数据管线的壁垒与边界
│   ├── 04_vertical_track_score_sheet.md   垂直赛道打分表
│   └── 05_hardtech_startup_risk_list.md   硬科技创业死亡风险清单
│
├── 04-vertical_case_study/                ← ✨场景案例研究（骨架，正文迭代中）
│   ├── case_01_industrial_inspect.md      工业检测/巡检机器人
│   └── case_02_workshop_robot.md          车间操作机器人
│
└── 05-mvp_practice/                       ← ✨课后 MVP 练习（骨架，代码迭代中）
    ├── README.md
    └── minimal_ik_dataset_pipeline_demo.py
```

---

## 🧭 四类读者，四条路径

<details open>
<summary><strong>🅰️ 工科跨 AI 实习生</strong></summary>

从 [4天从汽车到机器狗](00-getting-started/4天从汽车到机器狗.md) 开始 → 然后读 [AI工具协同作战](01-internship-toolchain/AI工具协同作战-企业级项目落地方法论.md) 学怎么用 AI 提高效率 → 需要搭 GitHub 时看 [GitHub 完全指南](01-internship-toolchain/GitHub从零发布完全指南.md)
</details>

<details>
<summary><strong>🅱️ 机器人 / 自动驾驶研发</strong></summary>

直奔 [VR遥操管线](02-vla-data-pipeline/从手到数据-VR遥操管线到底在跑什么.md)、[真机遥操第一天](02-vla-data-pipeline/真机遥操第一天-机器人动了也拧了.md)、[真机遥操第二天](02-vla-data-pipeline/真机遥操第二天-夹爪信号去哪了.md) 和 [openpi 数据格式笔记](02-vla-data-pipeline/给机器人基础模型准备教材-openpi数据格式学习笔记.md) → 关注每篇文末的「汽车-机器人对照表」→ 整车-机器人控制对照手册规划中（实习期工程笔记，后续进 02）
</details>

<details>
<summary><strong>🅲️ 校招求职</strong></summary>

看 [AI工具协同作战](01-internship-toolchain/AI工具协同作战-企业级项目落地方法论.md) 证明工程落地能力 → 看 [真机遥操第一天](02-vla-data-pipeline/真机遥操第一天-机器人动了也拧了.md) 证明排障与系统思维 → 看 [成长自述](03-post_intern_iterate/00_my_growth_journey.md) 和 [行业幻觉](03-post_intern_iterate/02_industry_myth_and_fake_demand.md) 了解这个人的认知边界与迭代习惯 → README 底部 Roadmap 证明持续产出能力 → 简历引用本仓库作为 PoW（工作量证明）
</details>

<details>
<summary><strong>🅳️ 硬科技爱好者 / 早期创业者</strong></summary>

从 [赛道拆解](03-post_intern_iterate/01_industry_track_decompose.md) 和 [死亡风险清单](03-post_intern_iterate/05_hardtech_startup_risk_list.md) 看产业判断框架 → [场景案例](04-vertical_case_study/) 看具体场景推演 → 记住这些内容都标注了瑕疵与信息缺口，是思路参考，不是行业报告
</details>

---

## 🛠 技术栈

```
具身智能：SLAM · MPC · A* · π₀.₅ · LeRobot v3.0 · OpenPI · Isaac Sim
车辆控制：CAN 总线 · PID · ECU MAP 标定 · IMU · 多传感器融合
LLM/RAG：LangChain · 向量检索 · 语义分块 · Agent 四种范式 · Prompt Engineering
工程工具：Git · GitHub Actions · FastAPI · WebSocket · Docker · Python
数据管线：UDP · HDF5 · Parquet · 分位数归一化 · 逆运动学(IK) · URDF · 真机遥操
```

---

## 🚀 Roadmap

| 阶段 | 内容 | 状态 |
|------|------|------|
| **已完成** | 4天入门日记、GitHub 指南、AI 工具协同方法论 | ✅ |
| **已完成** | VR 遥操管线全链路、openpi 数据格式学习笔记、IK v2 重构实录 | ✅ |
| **已完成** | 真机遥操第一天（链路跑通 + 构型失控排查实录） | ✅ |
| **进行中** | 阶段②目录骨架：成长自述/赛道拆解/伪需求/壁垒复盘/打分表/风险清单/场景案例/MVP（正文自 8.26 ARE 展会起逐篇填充） | 🔄 |
| **短期** | 机器狗实操笔记（SLAM 建图调参、MPC 步态力控、A* 路径规划踩坑，实习期工程笔记 → 00-02） | 📋 |
| **短期** | 企业级 RAG 落地（巡检知识库原型、分块策略对比、检索评测） | 📋 |
| **中期** | 整车-机器人控制对照手册（ECU↔LLM、ESP↔力控、CAN↔WebSocket） | 📋 |
| **中期** | 真机数据采集闭环（构型失控后续：硬件修复 → 拿方块正式数据集 → LeRobot 训练数据交付） | 📋 |
| **长期** | 阶段②正文成体系：从工程笔记到产业认知的完整成长轨迹 | 💡 |
| **长期** | 《汽车工程师转具身智能学习路线图》完整版 | 💡 |

---

## 👤 作者

**keywassion（梓华）** — 汽车工程大三，更确切的说是汽车服务工程，具身智能研发助理实习生。

大学挂过高数、线代课本没怎么翻过。但有一个不谦虚的优点：**给我一个东西，我能从底层把它拆明白。** 三个月前连「大模型」和「大数据」都分不清，现在在搭 VLA 数据管线，也会把自己犯过的错一条条写下来。

> 成绩单不反映动手能力。贪玩的人往往好奇心重，坐不住的人往往动手能力强。这个行业变得太快——真正稀缺的不是「什么都会的人」，是「能快速学会任何东西的人」。而比学得快更难得的，是敢承认自己曾经想错，并把它留在记录里。

- 📧 1873733846@qq.com
- 🐙 [github.com/hua0467](https://github.com/hua0467)

---

## 📄 其他

- **License**：[MIT](LICENSE) — 代码和文档都可以自由使用，署名即可
- **更新日志**：[CHANGELOG.md](CHANGELOG.md)
- **纠错/交流**：提 Issue 或直接发邮件。被纠错 = 白嫖了一节课，不亏。
- **关于 03-05 目录**：均为后期迭代产物（自 2026.8.26 ARE 展会起持续更新），不代表 8.26 之前的认知水平；每个文件头部有标记，正文未填充前只有提纲，请勿把提纲当结论。

---

<p align="center"><em>持续更新中 | 2026.08 | 一个非典型学生的非典型学习记录</em></p>
