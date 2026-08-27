# 2026-08-24 环境配置日志

> 目标：把 4090 训练主机配成可复现的机器人模仿学习训练环境。

## 1. 确认 GPU

```bash
nvidia-smi
```

| 项 | 值 |
|---|---|
| GPU | RTX 4090 |
| 显存 | 49140MiB（约 48GB） |
| 驱动 | CUDA 13.2 |

## 2. 安装 Miniforge3

- 安装 Miniforge3（conda 26.5.3）
- **坑 1**：装完 `conda` 命令找不到——miniforge3 没进 PATH
- 解决：

```bash
~/miniforge3/bin/conda init bash
source ~/.bashrc
```

## 3. 创建 Python 环境

```bash
conda create -n robot python=3.10 -y
conda activate robot
```

## 4. 安装 PyTorch（CUDA 版）

```bash
pip install torch==2.6.0 torchvision torchaudio --index-url https://download.pytorch.org/whl/cu124
```

验证 CUDA 可用：

```python
import torch
print(torch.cuda.is_available())  # True
```

## 5. 安装 LeRobot（源码安装，重要）

pip 直装踩坑（见 [issues_and_solutions.md](../issues_and_solutions.md) 问题 2/3），最终路线：

```bash
git clone https://github.com/huggingface/lerobot.git
cd lerobot
git fetch --tags
git checkout v0.4.4
pip install -e .
```

- pip 装的 0.4.4 只有库文件，**没有训练脚本模块**（训练脚本在源码 `src/` 目录下）
- 源码主分支要求 Python >= 3.12，与本机 Python 3.10 不兼容 → 切到 v0.4.4 标签
- 训练脚本模块名是 `lerobot.scripts.lerobot_train`（不是 `lerobot.scripts.train`）

## 6. 下载官方示例数据集

```bash
huggingface-cli download lerobot/aloha_sim_insertion_human --repo-type dataset --local-dir ~/lerobot_data/aloha_sim_insertion_human
```

| 属性 | 值 |
|---|---|
| episode | 50 |
| 总帧数 | 25000 |
| 动作维度 | 14 |
| 视觉 | 顶部摄像头视频 |
| 帧率 | 50 fps |

## 7. 环境就绪清单

- [x] GPU 可见，显存 48GB
- [x] conda 环境 `robot`（Python 3.10）
- [x] PyTorch 2.6.0+cu124，CUDA 可用
- [x] LeRobot v0.4.4 源码安装
- [x] 官方示例数据集就位
- [x] 预训练权重手动就位（resnet18-f37072fd.pth，见问题 7）
- [x] 视频解码回退 pyav（见问题 8）
