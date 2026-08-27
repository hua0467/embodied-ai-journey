# 2026-08-24 训练日志：10 步验证

> 目的：用最小代价验证训练链路整条通——脚本能起、数据能读、GPU 能吃、checkpoint 能存。10 步跑通再放大步数。

## 训练命令

```bash
WANDB_MODE=disabled python -m lerobot.scripts.lerobot_train \
  --policy.type=act \
  --env.type=aloha \
  --dataset.repo_id=~/lerobot_data/aloha_sim_insertion_human \
  --policy.device=cuda \
  --policy.push_to_hub=false \
  --dataset.video_backend=pyav \
  --output_dir=~/lerobot_output/act_test_10 \
  --steps=10
```

## 遇到的问题

- 模块名错误：`lerobot.scripts.train` → 实际是 `lerobot.scripts.lerobot_train`（`--help` 查出）
- `policy.push_to_hub` 默认 True → 报 `ValueError: 'policy.repo_id' argument missing` → 加 `--policy.push_to_hub=false`
- 缺 `gym_aloha` 仿真依赖 → `pip install gym-aloha`
- ResNet18 预训练权重官方源 SSL 失败 → 手动下载 45MB 权重到 `~/.cache/torch/hub/checkpoints/`
- torchcodec 缺 FFmpeg 共享库（libavutil.so.60 等）→ `--dataset.video_backend=pyav`

详见 [issues_and_solutions.md](../issues_and_solutions.md)。

## 成功输出

- 训练启动正常，无报错
- checkpoint 保存成功
