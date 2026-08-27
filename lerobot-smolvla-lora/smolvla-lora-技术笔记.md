# SmolVLA LoRA 微调技术笔记（支撑文档）

> 本文是《同一个评估脚本，结论从 -32.6% 翻到 +37.8%》的技术细节支撑，包含完整命令、超参、踩坑全表与评估方法学。环境：Ubuntu 22.04 / RTX 4090 48GB / Python 3.10 / PyTorch 2.6.0+cu124 / LeRobot v0.4.4（src 布局，`PYTHONPATH=~/lerobot/src`）。

## 1. 完整训练命令（LoRA 微调 SmolVLA）

```bash
cd ~/lerobot
WANDB_MODE=disabled PYTHONPATH=~/lerobot/src:$PYTHONPATH python -u src/lerobot/scripts/lerobot_train.py \
    --policy.type=smolvla \
    --peft.method_type=lora \
    --peft.r=16 \
    --peft.target_modules=all-linear \
    --policy.pretrained_path=lerobot/smolvla_base \
    --policy.load_vlm_weights=true \
    --policy.repo_id=local-smolvla-lora \
    --policy.push_to_hub=false \
    --dataset.repo_id=~/lerobot_data/<数据集路径> \
    --dataset.video_backend=pyav \
    --batch_size=4 \
    --steps=2000 \
    --output_dir=~/embodied_ai_phase2/task3_smolvla_lora
```

**关键点**：不要加 `--policy.use_peft=true`（那是推理时挂载已有适配器用的，会报 `Can't find 'adapter_config.json'`）。新建 LoRA 只传 `--peft.*` 参数。

## 2. 两次训练结果

| 数据集 | 可训参数 | 步数 | loss 轨迹 | 纯训练耗时 | 备注 |
|---|---|---|---|---|---|
| aloha_sim_insertion_human（官方，50eps/25000帧） | 11.5M（总 462M 的 2.5%） | 2000 | 0.096→0.846 | ~4 分钟 | 后期反弹，过拟合 |
| aloha_static_coffee（替身，50eps/55000帧，含语言指令） | 同上 | 2000 | 0.982→0.108 | 8 分 42 秒 | 单调收敛 |

## 3. 踩坑全表（12 个）

| # | 现象 | 根因 | 解决 |
|---|---|---|---|
| 1 | `ModuleNotFoundError: lerobot.common` | v0.4.4 src 布局，模块路径已改 | PYTHONPATH 指到 src，导入用 `lerobot.datasets` |
| 2 | `dataset.meta['total_actions']` TypeError | meta 是对象 | 属性访问 `dataset.meta.total_actions` |
| 3 | 训练结束推 Hub 报 401 | 默认行为 | `--policy.repo_id` 占位 + `--policy.push_to_hub=false` |
| 4 | `Can't find adapter_config.json` | `use_peft=true` 触发"加载已有适配器"分支 | 训练只传 `--peft.method_type=lora --peft.r=16 --peft.target_modules=all-linear` |
| 5 | hf-mirror 下载 VLM 大文件失败 | 镜像对大文件不稳定 | `unset HF_ENDPOINT` 直连 |
| 6 | 找不到训练脚本 | 入口在源码 scripts 目录 | `src/lerobot/scripts/lerobot_train.py` |
| 7 | draccus 报 `type` 字段非法 | HF 新版 config.json 新增 `type` 字段 | 临时副本 `pop("type")` 再 draccus.parse，不动缓存 |
| 8 | `All image features are missing from the batch` | config 默认键 camera1/2/3 ≠ 数据集键 observation.images.top | 用 `dataset_to_policy_features(dataset.meta.features)` 覆盖 config 的 input/output_features |
| 9 | `SmolVLAPolicy` 无 `predict()` | v0.4.4 API | processor 管线：`preprocessor(obs+task) → policy.predict_action_chunk(batch)` |
| 10 | 模型加载 23s（HF 网络重试） | hf_hub 每次 HEAD 请求 | `HF_HUB_OFFLINE=1`，降到 3.2s |
| 11 | hf_hub 库在镜像端点下载反复失败 | 库的元数据回落 bug | 镜像直链 + curl 并行下载（1.57GB） |
| 12 | 预测 vs 真值距离"变差 32.6%" | 模型输出在归一化空间、GT 在原始空间 | 用 `meta/stats.json` 的 mean/std 反归一化后比较 → 实际改善 37.8% |

## 4. 评估方法学（坑 12 的完整解法）

`predict_action_chunk` 输出的是归一化空间的预测动作（训练时数据按 mean/std 标准化）。与原始空间真值比较前必须反归一化：

```python
import json, numpy as np

with open("meta/stats.json") as f:
    stats = json.load(f)
mean = np.array(stats["action"]["mean"])   # 按数据集实际键名
std  = np.array(stats["action"]["std"])

# pred_norm: (1, 50, action_dim) 模型输出；取首步
pred_raw = pred_norm[0, 0] * std + mean
dist = np.linalg.norm(pred_raw - gt_raw)  # gt_raw 为原始空间真值
```

- 首版（未反归一化）：平均距离 0.854 → 1.132（"变差 32.6%"）
- 修正后：平均距离 0.854 → **0.531（改善 37.8%）**
- 20 个随机帧，数据集内指令，动作块首步，L2 距离，原始动作空间

## 5. 微调前后推理对比（15 组：5 帧 × 3 指令）

| 指标 | base | LoRA 微调后 |
|---|---|---|
| 模型加载时间 | 3.30s | 3.39s（HF_HUB_OFFLINE=1） |
| 显存增量 | 0.86 GB | 0.91 GB（双模型峰值 1.83 GB） |
| 单次推理时间 | 86–274ms | 147–160ms（LoRA 增 ~60ms） |
| 动作范数（归一化空间，50 步 chunk） | 9.6–32.6 | 15.0–28.3 |

- 微调前后动作差异 L2 = 18.4–36.5，输出分布显著偏移
- 数据集内长指令下 LoRA 输出更贴近演示动作；"do nothing"等无关指令下两模型差异最大——指令文本确实在影响输出，且微调改变了这一映射

## 6. 数据下载（替身数据集）

aloha_static_coffee（1.57GB，v3.0 格式，50 eps / 55000 帧，含语言指令）。HF 库在镜像端点下有元数据回落 bug，直接用镜像直链 + curl 并行下载最稳。下载后验证：`meta/info.json` 的 `codebase_version` 应为 v3.0；`meta/tasks.parquet` 含 task 字段。

## 7. 结论与边界

- LoRA（r=16，all-linear）在 4090 上以 2.5% 可训参数完成 SmolVLA 微调，2000 步 8 分 42 秒
- "预测 vs 真值"距离下降 37.8% 是离线代理指标，不是任务成功率；且替身数据可能已在预训练集内，收益不可直接外推到自采数据
- 自采 VR 数据到位后：换 `--dataset.repo_id` 重跑本管线即可
