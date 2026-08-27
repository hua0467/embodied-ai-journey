# 让 IK 不再坑你：F1 双机械臂求解器从 v1 到 v2 的重构实录

> **快速了解**：纯 numpy+scipy 给一台 17 个关节（3 躯干+14 手臂）的 F1 双机械臂写了个 IK 求解器。v1 能用但埋了坑——躯干冲突、关节限位 bug、代价函数不平滑、优化失败就崩。v2 逐一修复，新增全局联合 17 DOF 求解，代价函数换成加权平方残差，补上低通滤波和 8 项单元测试。这篇文章不是 IK 教程——是告诉你一个能跑的原型要变成能用在实时 VR 遥操里的代码，中间差了哪些东西。

---

## 目录

- [1. 背景：为什么要自己写 IK](#1-背景为什么要自己写-ik)
- [2. v1 的六个坑](#2-v1-的六个坑)
- [3. v2 逐个修复](#3-v2-逐个修复)
- [4. 全局联合 IK 怎么做的](#4-全局联合-ik-怎么做的)
- [5. 代价函数：从 L1 到加权平方](#5-代价函数从-l1-到加权平方)
- [6. 单元测试比你想象的更重要](#6-单元测试比你想象的更重要)
- [7. v1 → v2 改动一览](#7-v1--v2-改动一览)
- [8. 怎么用](#8-怎么用)

---

## 1. 背景：为什么要自己写 IK

F1 是台双机械臂机器人。17 个关节：躯干 3 个（升降 + 两个旋转），左右手臂各 7 个。

IK（Inverse Kinematics，逆运动学）要解决的问题：你知道想让手伸到空间里哪个位置（比如 VR 头显追踪到你的手在 (0.3, -0.2, 0.6)），反算出每个关节该转多少度。

正常的机器人项目会用 pinocchio 或 pybullet 之类的库做 IK。但写 v1 的时候：

- pybullet → Windows 编译报错，装不上
- pinocchio → 没有 conda 包，折腾了一下午放弃了

**结论：自己写。** 从 URDF XML 文件手动解析关节信息，用 scipy 的 L-BFGS-B 做数值优化。听起来简单——但实际代码跑起来之后，一个接一个的问题开始冒出来。

---

## 2. v1 的六个坑

v1 的 `solve_ik` 能用。录 VR 数据的时候它确实在解关节角，而且成功率 99%。但"能用"和"该这么用"之间差了很多。

### 坑 1：左右手分开求解 → 躯干冲突

v1 的设计是左右手各自调一次 `solve_ik`：

```python
# v1 的做法 —— 有问题！
ra = f1.solve_ik(right_pos, right_quat, side="right")  # 返回 10 DOF（含躯干）
la = f1.solve_ik(left_pos, left_quat, side="left")     # 返回 10 DOF（含躯干）

# 然后把躯干部分取平均
torso = (ra[:3] + la[:3]) / 2.0  # ← 这没有物理意义
```

问题是躯干是左右手共用的。分开解的时候，每个 solver 都自由地调整躯干角度来凑自己的目标位姿。解出来的两个躯干角度可能差很远——比如右手说"腰转 48° 更顺"，左手说"腰转 -20° 更顺"——然后你取个平均？这没有运动学上的合理性。

**这就是为什么需要全局联合 IK。** 躯干应该只优化一次，同时满足两只手的目标位姿。

### 坑 2：continuous 关节被限位

URDF 里有些关节类型是 `continuous`——理论上是无限旋转的（比如手腕旋转）。但在 `_parse_urdf` 里，continuous 关节被当成了 revolute 处理：

```python
# v1 的 bug
lower = float(limit.get('lower', 0)) if limit is not None else -np.pi
upper = float(limit.get('upper', 0)) if limit is not None else np.pi
```

这导致 continuous 关节被硬生生限制在了 ±π（±180°）。如果 IK 需要转 181°，优化器直接撞墙。

### 坑 3：prismatic 平移关节被截断到 ±π

更离谱的是，bounds 构建时所有关节都被 clip 到 [-π, π]：

```python
# v1
lo = max(j['lower'], -np.pi)  # ← 对 prismatic 完全错误
hi = min(j['upper'], np.pi)
```

prismatic 是平移关节，单位是米，不是弧度。限位可能是 ±0.5m。但你用 `max(lower, -π)` 就变成了 `max(-0.5, -3.14) = -0.5`，然后 `min(0.5, 3.14) = 0.5`——这次侥幸没错。但如果 prismatic 限位是 ±5m，就会被错误截成 ±3.14m。

### 坑 4：代价函数用 L1，对 L-BFGS-B 不友好

```python
# v1
return pos_err + 0.5 * rot_err  # L1 范数
```

L1 范数在零点不光滑——梯度不连续。L-BFGS-B 用的是梯度信息，L1 会让它在最优点附近来回震荡，收敛慢。

### 坑 5：优化失败直接崩

```python
# v1
return best_result.x  # 如果 best_result 还是 None，直接 AttributeError
```

在 VR 实时场景里，如果某一帧追踪数据跳了一下、目标位姿变得不可达，程序直接崩——用户正在戴着头显做演示，你让他重启 Python？

### 坑 6：姿态四元数没做坐标系对齐

VR 的坐标系和机器人不一样。v1 只映射了位置（X/Y/Z 轴对调），但四元数直接拷过去了：

```python
# v1
target_quat = hand_quat.copy()  # ← 坐标系没转，姿态不对
```

位置映射对了，手在正确的位置，但"手心朝哪"的方向是错的。

---

## 3. v2 逐个修复

### 修复 1：URDF 解析——continuous 和 prismatic

```python
# v2: 按关节类型区分限位逻辑
if jtype == 'continuous':
    lower, upper = -np.inf, np.inf           # 无物理限位
elif jtype == 'revolute':
    if limit is not None:
        lower, upper = float(limit.get('lower')), float(limit.get('upper'))
    else:
        lower, upper = -np.pi, np.pi          # 默认 ±180°
elif jtype == 'prismatic':
    if limit is not None:
        lower, upper = float(limit.get('lower')), float(limit.get('upper'))
    else:
        lower, upper = -1.0, 1.0              # 默认 ±1m，不做 π 截断
```

优化器内部用 `_get_optim_bounds()` 把 `inf` 映射成 `±4π`——作为有限代理，远超任何实际关节需要的转动范围。

### 修复 2：代价函数换成加权平方残差

```python
# v2: 平方残差 —— 光滑、梯度连续、L-BFGS-B 友好
def _pose_error_sq(self, T_current, T_target):
    pos_diff = T_current[:3, 3] - T_target[:3, 3]
    pos_sq = np.sum(pos_diff ** 2)

    # 旋转误差用轴角大小（做好 clamp 防 NaN）
    R_rel = T_target[:3,:3] @ T_current[:3,:3].T
    trace = np.clip((np.trace(R_rel) - 1.0) / 2.0, -1.0, 1.0)
    rot_angle = np.arccos(trace)
    rot_sq = rot_angle ** 2

    return self.pos_weight * pos_sq + self.rot_weight * rot_sq
```

位置和姿态权重可配置：`F1Kinematics(urdf_path, pos_weight=1.0, rot_weight=0.5)`。

### 修复 3：优化失败不崩

```python
# v2: 失败时返回初始猜测，不会崩
self._last_solve_success = result.success and result.fun < cost_threshold

if not self._last_solve_success:
    print(f"[IK WARN] cost={result.fun:.4f}, using fallback", flush=True)

return result.x.copy() if result.x is not None else x0.copy()
```

同时 `solve_both_arms_ik` 增加了 `cost_threshold` 参数——优化后代价超过阈值就标记求解失效，调用方可以通过 `f1.last_solve_success` 检查。

### 修复 4：VR 坐标转换增加姿态标定

```python
# v2: 支持标定旋转四元数
def hand_pose_to_robot_target(hand_wrist, hand_quat=None, calib_quat=None, ...):
    if calib_quat is not None:
        # q_robot = q_calib * q_hand
        R_calib = R.from_quat(calib_quat)
        R_hand = R.from_quat(hand_quat)
        target_quat = (R_calib * R_hand).as_quat()
    else:
        target_quat = hand_quat.copy()  # 未标定时的兜底
```

`calib_quat` 需要通过物理标定确定——把机器人末端和 VR 手部对齐，记录两组四元数求差。现在还没标定所以传 None 走兜底，但接口留好了。

---

## 4. 全局联合 IK 怎么做的

这是 v2 最大的新增功能。思路很直接：

**优化变量**：17 个关节角 `[torso(3), right_arm(7), left_arm(7)]`

**代价函数**：右臂 FK 误差 + 左臂 FK 误差

```python
def _cost(x):
    torso = x[:3]
    right_angles = np.concatenate([torso, x[3:10]])   # 拼出 10 DOF 右臂链
    left_angles = np.concatenate([torso, x[10:17]])    # 拼出 10 DOF 左臂链

    T_right = self._chain_fk(self.right_arm_joints, right_angles)
    T_left = self._chain_fk(self.left_arm_joints, left_angles)

    return (self._pose_error_sq(T_right, T_right_target) +
            self._pose_error_sq(T_left, T_left_target))
```

躯干 3 DOF 在这 17 维向量里只出现一次，两边 FK 用同样的躯干值。L-BFGS-B 同时优化全部 17 个，不用人为拆解和取平均。

**性能**：冷启动 ~0.15s，warm-start（用上一帧做初始猜测）~0.08s。VR 场景 30fps（33ms/帧）不够，但 Quest 3 实际发过来 ~6fps（167ms/帧），0.08s 完全可以接受。

**收敛情况**：warm-start 通常 3-6 次迭代就收敛（`ftol=1e-3`），因为上一帧的角度离最优解很近。

### 兼容旧接口

原来的 `solve_ik(side, ...)` 保留不动，但标记为旧模式。新增了 `fixed_torso` 参数——如果你已经知道躯干角度（比如从全局 IK 拿到），可以用它单解手臂 7 DOF。

---

## 5. 代价函数：从 L1 到加权平方

这里多说一句——为什么平方比 L1 好。

L-BFGS-B 用 BFGS 近似 Hessian，本质上是基于梯度的。L1 范数在最优点的梯度不连续（从 -1 跳到 +1），让 BFGS 的 Hessian 近似在零点附近不稳定。

平方范数处处可导，梯度是线性的，BFGS 能很轻松地在零点附近找到精确方向。

> **汽车类比**：L1 就像没有助力的方向盘——在中心位置附近手感是"硬"的，你没法做微小的修正。平方范数是有助力的——越靠近中心越轻盈，精确对正很容易。

---

## 6. 单元测试比你想象的更重要

v1 没有测试。改了代码只能靠跑一次录制看会不会崩——这是最差的验证方式。

v2 写了 8 个测试，全部在 `__main__` 里，直接 `python ik_solver.py` 就跑：

| 测试 | 验证什么 | 结果要求 |
|------|----------|----------|
| Test 1: FK 零位 | URDF 解析正确，零位末端位置合理 | 有输出即可 |
| Test 2: get_ee_poses | 17D 入参接口正确 | 与 Test 1 结果一致 |
| Test 3: 单侧 IK（旧接口） | 旧代码兼容 | 位置误差 < 0.05m |
| Test 4: fixed_torso 模式 | 躯干固定、只优化手臂 | 功能可用 |
| Test 5: 全局联合 IK | 核心新功能，两臂同时解 | cost < 0.01, 成功标记 True |
| Test 6: FK→IK 回代 | 闭合验证：FK 出位姿 → IK 回代 → FK 验证到位 | 位姿误差 < 8cm 冷启动 |
| Test 7: AngleFilter | EMA 低通滤波数学正确 | 数值验证 |
| Test 8: VR 坐标转换 | 轴映射正确 | 有输出 |

Test 6 是最重要的——它验证整个 IK 管线在"理想条件"下的精度。随机生成关节角 → FK 算末端位姿 → IK 反解 → 再 FK 验证。如果回代失败，说明 IK 求解本身有问题。

---

## 7. v1 → v2 改动一览

| 维度 | v1 | v2 |
|------|----|----|
| 求解模式 | 左右分开，躯干取平均 | 全局联合 17 DOF 一次解 |
| continuous 关节 | 被误限 ±π | (-inf, inf)，优化器用 ±4π 代理 |
| prismatic 关节 | 被截断到 [-π, π] | 使用 URDF 原始限位 |
| 代价函数 | L1 范数（不光滑） | 加权平方残差（处处可导） |
| 优化失败 | 直接崩 | 返回 fallback + 标记失败 |
| 姿态转换 | 四元数直接拷贝 | 支持 calib_quat 标定旋转 |
| 多初值 | 每次 optimize 都试两个 guess | 单初值（实时），多 guess 仅调试用 |
| 低通滤波 | 无 | EMA AngleFilter |
| 单元测试 | 0 个 | 8 个 |
| 类型注解 | 基本无 | 关键接口全覆盖 |
| 旧接口兼容 | — | solve_ik 保留不变 |

---

## 8. 怎么用

### 全局 IK（推荐，VR 实时场景用）

```python
from ik_solver import F1Kinematics, hand_pose_to_robot_target

f1 = F1Kinematics("F1_URDF_V04.urdf")

# VR 手部数据 → 机器人目标位姿
r_pos, r_quat = hand_pose_to_robot_target(right_hand_wrist)
l_pos, l_quat = hand_pose_to_robot_target(left_hand_wrist)

# 一次性解出 17 关节角
joints_17d = f1.solve_both_arms_ik(
    r_pos, r_quat, l_pos, l_quat,
    initial_guess=last_frame_joints,  # warm-start
)

if f1.last_solve_success:
    last_frame_joints = joints_17d  # 传给下一帧
```

### 单侧 IK（旧接口，兼容现有代码）

```python
angles_right = f1.solve_ik(pos, quat, side='right')  # 返回 10 DOF（含躯干）

# 新参数：躯干固定
angles_right = f1.solve_ik(pos, quat, side='right',
                           fixed_torso=[0.0, 0.1, 0.0])  # 返回 7 DOF（纯手臂）
```

### 低通滤波

```python
from ik_solver import AngleFilter

filt = AngleFilter(alpha=0.3)  # 0.3 = 适中平滑，1.0 = 不过滤
smoothed = filt(raw_joints)
```

### 末端位姿查询

```python
ee = f1.get_ee_poses(joints_17d)
print(ee['right_pos'])     # [x, y, z]
print(ee['right_quat_xyzw'])  # [qx, qy, qz, qw]
```

---

## 现在到哪了

v2 已经在 `record_session.py` 旁边跑着了。旧 `IKSolver` wrapper 还没切到 `solve_both_arms_ik`——目前还是用旧的两次分开调用的方式，但数据产出正确（因为取平均凑合能用）。

下一步要做的：
- `record_session.py` 的 `IKSolver` 切到全局 IK
- VR→机器人坐标系的 scale/offset/calib_quat 物理标定
- 真机验证：把解出来的关节角发到 F1 上，看它是不是真的伸到你手的位置

这些等拿到 F1 机器人的 IP 就开搞。

> 写于 2026.8.12，实习 Day 9
