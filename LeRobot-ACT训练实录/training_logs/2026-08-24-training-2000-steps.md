# 2026-08-24 训练日志：2000 步

> 目的：完整跑一轮训练，拿到可分析的 loss 曲线与 checkpoint，验证在 4090 上能独立完成机器人策略训练。

## 训练命令

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

- 耗时：80 秒
- 速度：约 25 step/s
- checkpoint：`~/lerobot_output/act_test_2000/checkpoints/002000`
- 模型参数量：51,613,582（约 52M）

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

## 结果分析

1. **loss 单调下降、无振荡**：2000 步对 ACT 只是热身，1.27 离收敛还远，但趋势完全正常——链路验证的目的达到
2. **速度可支撑规模训练**：25 step/s 意味着 5 万步约 33 分钟；正式训练以数万步计，这台机器完全扛得住
3. **checkpoint 机制正常**：002000 落盘，后续可从任意步数续训
4. **52M 参数量符合 ACT 量级**：对单机后训练友好

## 下一步

换成自己采的 VR 遥操作真机数据，重复这条链路——数据格式对齐见 [openpi 数据格式笔记](../../02-VLA数据管线-从遥操到真机/给机器人基础模型准备教材-openpi数据格式学习笔记.md)。
