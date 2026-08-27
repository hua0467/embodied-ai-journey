# 2026-08-24 训练日志：500 步

> 目的：放大到 500 步，观察 loss 是否正常下降、速度是否可接受。

## 训练命令

```bash
WANDB_MODE=disabled python -m lerobot.scripts.lerobot_train \
  --policy.type=act \
  --env.type=aloha \
  --dataset.repo_id=~/lerobot_data/aloha_sim_insertion_human \
  --policy.device=cuda \
  --policy.push_to_hub=false \
  --dataset.video_backend=pyav \
  --output_dir=~/lerobot_output/act_test_500 \
  --steps=500
```

## 训练结果

- 耗时：21 秒
- 速度：约 25 step/s

| step | loss |
|---|---|
| 200 | 7.315 |
| 400 | 3.048 |

- checkpoint 保存成功
- loss 从 7.3 降到 3.0，趋势正常

## 结论

链路稳定、速度可接受 → 放大到 2000 步。
