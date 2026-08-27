# 训练命令全集（2026-08-24）

> 按执行顺序排列，训练机主目录以下文 `~/` 表示，全部可直接复制。每条命令上方有用途说明。

## 0. 环境检查

```bash
nvidia-smi
```

## 1. Miniforge3 安装后初始化（解决 conda 命令找不到）

```bash
~/miniforge3/bin/conda init bash
source ~/.bashrc
```

## 2. 创建 Python 3.10 环境

```bash
conda create -n robot python=3.10 -y
conda activate robot
```

## 3. 安装 PyTorch 2.6.0（CUDA 12.4 版）

```bash
pip install torch==2.6.0 torchvision torchaudio --index-url https://download.pytorch.org/whl/cu124
```

## 4. 源码安装 LeRobot v0.4.4

```bash
git clone https://github.com/huggingface/lerobot.git
cd lerobot
git fetch --tags
git checkout v0.4.4
pip install -e .
```

## 5. 下载官方示例数据集

```bash
huggingface-cli download lerobot/aloha_sim_insertion_human --repo-type dataset --local-dir ~/lerobot_data/aloha_sim_insertion_human
```

## 6. 补仿真依赖（env.type=aloha 必须）

```bash
pip install gym-aloha
```

## 7. 预训练权重手动下载（网络受限时）

ResNet18 权重 `resnet18-f37072fd.pth`（约 45MB）手动下载后放到：

```
~/.cache/torch/hub/checkpoints/resnet18-f37072fd.pth
```

## 8. 训练（10 步验证 → 500 步 → 2000 步）

```bash
# 10 步验证：最小代价确认链路全通
WANDB_MODE=disabled python -m lerobot.scripts.lerobot_train \
  --policy.type=act \
  --env.type=aloha \
  --dataset.repo_id=~/lerobot_data/aloha_sim_insertion_human \
  --policy.device=cuda \
  --policy.push_to_hub=false \
  --dataset.video_backend=pyav \
  --output_dir=~/lerobot_output/act_test_10 \
  --steps=10

# 500 步：观察 loss 趋势与训练速度
# （同上，替换 --output_dir=~/lerobot_output/act_test_500 与 --steps=500）

# 2000 步：完整一轮
# （同上，替换 --output_dir=~/lerobot_output/act_test_2000 与 --steps=2000）
```

## 关键参数说明

| 参数 | 作用 |
|---|---|
| `WANDB_MODE=disabled` | 训练机不连 W&B，关闭实验跟踪 |
| `--policy.push_to_hub=false` | 训练完不自动上传 HF Hub（默认 True，会报缺 repo_id） |
| `--dataset.video_backend=pyav` | 视频解码用 pyav（默认 torchcodec 依赖系统 FFmpeg 共享库） |
| `--steps` | 训练总步数 |
| `--output_dir` | checkpoint 与日志输出目录 |
