# LeRobot ACT Training on RTX 4090

> **快速了解**：在一台 48GB 显存的 RTX 4090 训练机上，从零完成 Ubuntu + Miniforge + PyTorch + LeRobot v0.4.4 环境搭建，用官方示例数据集跑通 ACT（Action Chunking Transformer）模仿学习训练链路——2000 步 loss 从 7.315 降到 1.266，约 25 step/s，checkpoint 正常落盘。这是用我自己采集的 VR 遥操作数据训练 VLA 之前的链路验证：证明数据格式、训练脚本、GPU 环境整条线在自己手上跑得通。

## 目录

- [环境信息](#环境信息)
- [数据集](#数据集)
- [训练命令](#训练命令)
- [训练结果](#训练结果)
- [遇到的问题与解决方案](#遇到的问题与解决方案)
- [分日训练日志](#分日训练日志)
- [下一步计划](#下一步计划)

## 环境信息

| 项 | 值 |
|---|---|
| GPU | RTX 4090，显存 49140MiB（约 48GB） |
| 驱动 | CUDA 13.2 |
| 系统 | Ubuntu 22.04 LTS |
| Python | 3.10（conda 环境名 `robot`，Miniforge3 26.5.3） |
| PyTorch | 2.6.0+cu124 |
| LeRobot | v0.4.4（源码安装；pip 装的 0.4.4 没有训练脚本模块） |

## 数据集

官方示例数据集 `aloha_sim_insertion_human`（HuggingFace）：

| 属性 | 值 |
|---|---|
| episode | 50 |
| 总帧数 | 25000 |
| 动作维度 | 14（双臂 ALOHA，每臂 7 关节） |
| 视觉 | 顶部摄像头视频 |
| 帧率 | 50 fps |

本地路径：`~/lerobot_data/aloha_sim_insertion_human`（`~/` 表示训练机主目录）。

## 训练命令

2000 步完整训练（W&B 关闭，视频解码走 pyav）：

```bash
WANDB_MODE=disabled python -m lerobot.scripts.lerobot_train \
  --policy.type=act \
  --env.type=aloha \
  --dataset.repo_id=~/lerobot_data/aloha_sim_insertion_human \
  --policy.device=cuda \
  --policy.push_to_hub=false \
  --dataset.video_backend=pyav \
  --output_dir=~/lerobot_output/act_test_2000 \
  --steps=2000
```

## 训练结果

2000 步 loss 记录（每 200 步）：

| step | loss |
|---|---|
| 200 | 7.315 |
| 400 | 3.047 |
| 600 | 2.502 |
| 800 | 2.209 |
| 1000 | 2.000 |
| 1200 | 1.803 |
| 1400 | 1.648 |
| 1600 | 1.492 |
| 1800 | 1.380 |
| 2000 | 1.266 |

- 训练速度：约 **25 step/s**（500 步 21 秒，2000 步 80 秒）
- checkpoint：`~/lerobot_output/act_test_2000/checkpoints/002000`
- 模型参数量：51,613,582（约 52M）

## 遇到的问题与解决方案

本次共踩 8 个坑（PATH、pip 缺脚本、版本兼容、模块名、默认参数、仿真依赖、预训练权重下载、视频解码），全部记录在 [issues_and_solutions.md](issues_and_solutions.md)——每个都含报错原文、原因分析和解决步骤。

## 分日训练日志

- [2026-08-24 环境配置](training_logs/2026-08-24-environment-setup.md)
- [2026-08-24 10 步验证训练](training_logs/2026-08-24-training-10-steps.md)
- [2026-08-24 500 步训练](training_logs/2026-08-24-training-500-steps.md)
- [2026-08-24 2000 步训练](training_logs/2026-08-24-training-2000-steps.md)

## 下一步计划

1. 用自己采的 VR 遥操作真机数据转成 LeRobot 格式，替换示例数据集再训
2. 对齐数据维度：真机数据目前缺夹爪维度，补齐后跑归一化统计（格式合同见 [openpi 数据格式笔记](../02-vla-data-pipeline/给机器人基础模型准备教材-openpi数据格式学习笔记.md)）
3. 拉大训练规模：2000 步只是链路验证，正式训练以万步计
4. 后续切换到 π₀.₅（openpi）做机器人基础模型后训练

---

*写于 2026.8.24*
