# 50分钟 Tutorial 讲授大纲（面向本次 Go2 作业）

## 讲授目标

这一讲的目标不是教会同学推导 PPO 公式，而是让他们真正掌握一条完整的 robot policy pipeline：

**MuJoCo XML -> Environment -> Observation / Action / Reward -> PPO Training -> Checkpoint -> Rollout -> Benchmark**

学生离开教室时，至少要能回答下面 6 个问题：

1. policy 的输入是什么，输出是什么？
2. 为什么 action 不是直接输出 torque？
3. actor 和 critic 为什么看到的信息不一样？
4. domain randomization 在哪里做，为什么做？
5. train / test / benchmark 分别在哪个脚本里？
6. 自己应该改哪里，不应该改哪里？

---

## 建议时间分配

### 0 - 5 min：作业目标与最终交付物

要讲清楚三件事：

- 这次作业的核心不是“从零写 PPO”
- 这次作业的核心是“理解并修改一个完整 pipeline”
- 最终提交物包括：
  - checkpoint
  - public eval JSON
  - demo video
  - short report

建议现场展示项目目录树，让学生一开始就建立全局结构感。

---

### 5 - 10 min：Colab 运行方式与可复现性

这一段只讲最必要的内容：

- 为什么要求 GPU runtime
- 为什么要固定版本 / pinned commit
- 为什么设置 `MUJOCO_GL=egl`
- 为什么设置 `JAX_DEFAULT_MATMUL_PRECISION=highest`
- 为什么这个作业要限制在 Colab 可承受的训练预算内

建议结论化表达：

> 我们不是在追求极限性能，而是在追求“所有人都能公平复现 baseline”。

---

### 10 - 18 min：MuJoCo / Playground / Brax 各自负责什么

这一段要讲清软件栈：

- **MuJoCo**：物理引擎与 XML 模型
- **MJX**：JAX 化、可批量并行的物理后端
- **MuJoCo Playground**：现成任务环境与训练接口
- **Brax PPO**：训练算法层

建议配一张最简单的 stack 图：

`XML model -> Playground env -> Brax PPO -> checkpoint -> rollout`

不要讲太多生态史，重点是让学生知道“每一层负责什么”。

---

### 18 - 27 min：重点讲 `go2_pg_env/joystick.py`

这是整堂课最重要的一段。

建议按下面顺序讲：

#### 1. Observation
先讲 actor observation：
- local linear velocity
- gyro
- projected gravity
- joint position error
- joint velocity
- last action
- command

再讲 critic privileged observation：
- 为什么训练时 critic 可以看到更多 simulator-only 信息
- 为什么部署时只保留 actor

这一段一定要把 “training-time information” 和 “deployment-time information” 分开讲清楚。

#### 2. Action
讲清楚：
- policy 输出的是 12 维 joint offset
- 最终 target = `default_pose + action_scale * action`
- actuator 再把 position target 转成 torque

学生只要理解这一层，后面很多 confusion 会自然消失。

#### 3. Reward
不要逐项背 reward，按三类讲：

- **Task terms**
  - tracking linear velocity
  - tracking yaw velocity

- **Stability terms**
  - orientation
  - vertical velocity
  - body angular velocity
  - termination

- **Smoothness / gait terms**
  - action rate
  - energy
  - feet slip
  - feet air time

这一段的目标不是记公式，而是理解“为什么这些项会塑造 gait”。

---

### 27 - 34 min：讲 `randomize.py` 与 two-stage curriculum

#### Domain randomization
重点讲：
- 摩擦
- 质量
- armature
- COM 偏移
- 初始姿态微扰

然后讲一句核心话术：

> 如果训练时永远只在一个完美 simulator 里学，部署时通常会因为模型误差而掉性能。

#### Two-stage curriculum
建议这么讲：

- **Stage 1**：先学会稳定走
  - smaller command range
  - weaker task difficulty

- **Stage 2**：再扩展能力与平滑性
  - larger command range
  - stronger smoothness / energy regularization

这一段要让学生明白：
curriculum 不是玄学，而是把任务难度按阶段展开。

---

### 34 - 41 min：讲 `train.py` / `test_policy.py` / `generate_public_rollout.py` / `public_eval.py`

建议用“职责划分”来讲。

#### `train.py`
负责：
- 载入 config
- 应用 stage config
- 调 PPO
- 保存 checkpoint
- 记录 progress / summary

重点提醒：
- checkpoint 不一定“最后一个最好”
- 这版代码已经修正为“导出 best eval checkpoint”

#### `test_policy.py`
负责：
- 恢复 checkpoint
- 跑 deterministic demo script
- 输出 video 与 demo summary
- 顺手算一次 benchmark metric

#### `generate_public_rollout.py`
负责：
- 使用标准 command script
- 产生统一格式 rollout bundle
- 给 benchmark 打分做输入

#### `public_eval.py`
负责：
- 从 rollout bundle 里读出指标
- 指标标准化
- 输出 composite score

这一段一定要强调：

> 训练 reward 和 benchmark metric 不是一回事。

---

### 41 - 46 min：现场 hands-on demo

最推荐的 live demo 顺序：

1. 运行 `inspect_env.py`
2. 展示 `joystick.py` 的 observation layout
3. 展示 `course_config.json` 里的 stage_1 / stage_2
4. 展示一个已经训练好的 checkpoint
5. 运行 `test_policy.py`，生成 demo video
6. 展示 `public_eval.json`

为什么不建议 live full training：
- Colab 第一次 JIT compile 有等待时间
- 50 分钟 lecture 容易被 compile 占掉节奏
- 恢复 checkpoint 更适合教学展示 pipeline

---

### 46 - 50 min：作业要求、评分和常见坑

建议重点提醒：

#### 作业允许改的地方
- `joystick.py`
- `randomize.py`
- `course_config.json`

#### 不建议改的地方
- benchmark evaluator
- rollout bundle 字段
- checkpoint 恢复逻辑

#### 常见坑
- 把 benchmark 当训练 reward
- 只看最后 checkpoint，不看 best checkpoint
- 训练效果很好，但 public benchmark 命令脚本没跑过
- 只会跑 notebook，不理解每个脚本负责什么

最后 1 分钟给出一句收束：

> 这次作业最重要的收获，不是“刷出最高分”，而是你真正知道一条 robot policy 是怎么被训练、保存、恢复、评测、再送去实机候选的。

---

## 建议的 live demo 命令

### 展示环境结构
```bash
python inspect_env.py --stage-name stage_2
```

### 恢复最佳 checkpoint 并渲染 demo
```bash
python test_policy.py \
  --checkpoint-dir artifacts/run_baseline/best_checkpoint \
  --stage-name stage_2 \
  --output-dir artifacts/demo_bundle
```

### 生成公开 benchmark
```bash
python generate_public_rollout.py \
  --checkpoint-dir artifacts/run_baseline/best_checkpoint \
  --stage-name stage_2 \
  --output-dir artifacts/public_eval_bundle \
  --num-episodes 4 \
  --render-first-episode
```

### 打分
```bash
python public_eval.py \
  --rollout-npz artifacts/public_eval_bundle/rollout_public_eval.npz \
  --output-json artifacts/public_eval_bundle/public_eval.json
```
