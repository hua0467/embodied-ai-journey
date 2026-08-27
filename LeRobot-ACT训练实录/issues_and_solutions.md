# 问题与解决方案全记录（2026-08-24）

> 本次共 8 个问题，全部有报错原文、原因分析、解决步骤，按出现顺序排列。

## 问题 1：Miniforge 安装后 conda 命令找不到

- **现象**：安装 Miniforge3 后，终端输入 `conda` 提示 command not found
- **原因**：miniforge3 未加入 PATH
- **解决**：

```bash
~/miniforge3/bin/conda init bash
source ~/.bashrc
```

- **经验**：装 conda 类工具的最后一步永远是 init + 重开 shell；报 command not found 先查 PATH，别急着重装

## 问题 2：pip install lerobot 后没有训练脚本

- **现象**：`pip install lerobot` 装的是 0.4.4，但找不到训练脚本模块
- **原因**：pip 包只装库文件，训练脚本在源码仓库的 `src/lerobot/scripts/` 目录下
- **解决**：从 GitHub 克隆源码 → `pip install -e .` 源码安装
- **经验**：要跑官方脚本的项目，直接源码安装，别用 pip 成品包

## 问题 3：LeRobot 主分支要求 Python >= 3.12

- **现象**：源码安装报 Python 版本不满足
- **原因**：主分支已升级，与 Python 3.10 不兼容
- **解决**：`git fetch --tags` 拉取所有标签，`git checkout v0.4.4` 切兼容版本
- **经验**：clone 完先看 pyproject.toml 的 requires-python，不匹配就找历史 tag

## 问题 4：训练脚本模块名错误

- **错误名**：`python -m lerobot.scripts.train`
- **正确名**：`python -m lerobot.scripts.lerobot_train`
- **解决**：用 `--help` 查参数时发现真实模块名
- **经验**：不确定入口模块时，`ls src/lerobot/scripts/` 或 `--help` 最直接

## 问题 5：policy.push_to_hub 默认 True 导致报错

- **报错**：`ValueError: 'policy.repo_id' argument missing`
- **原因**：训练结束默认要把模型推到 HF Hub，但没给 repo_id
- **解决**：加 `--policy.push_to_hub=false`
- **经验**：默认行为里带"联网动作"的参数（push/upload/log），优先显式关掉

## 问题 6：缺少 gym_aloha 仿真依赖

- **报错**：`ModuleNotFoundError: No module named 'gym_aloha'`
- **解决**：`pip install gym-aloha`
- **经验**：env.type=aloha 即使只用数据集也要装对应仿真包——env 的注册在导入期就发生

## 问题 7：ResNet18 预训练权重下载失败

- **现象**：训练启动时下载预训练权重报 SSL 错误
- **原因**：PyTorch 官方源被网络拦截
- **解决**：手动下载 `resnet18-f37072fd.pth`（45MB）到 `~/.cache/torch/hub/checkpoints/`
- **经验**：训练机网络受限是常态，权重、模型、数据集都提前准备离线副本

## 问题 8：torchcodec 视频解码失败

- **现象**：默认视频后端 torchcodec 报缺 FFmpeg 共享库（`libavutil.so.60` 等）
- **原因**：系统缺 FFmpeg 共享库
- **解决**：加 `--dataset.video_backend=pyav`（环境里已有 av 库）
- **经验**：视频解码后端是高频坑，pyav 是更省心的默认选择

## 总结

8 个问题没有一个是算法问题——全是环境、版本、路径、默认参数。**这恰恰说明：训练链路能不能跑通，瓶颈不在模型，在工程细节。** 把这一层趟平，后面换数据、换策略（π₀.₅）都只是换参数。
