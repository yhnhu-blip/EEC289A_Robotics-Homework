# PPT 讲解思路（逐页设计）

下面是一套适合 50 分钟 tutorial 的 slide-by-slide 设计。  
建议总页数控制在 **12 - 14 页**，否则 live demo 时间容易被压缩。

---

## Slide 1：Title

**标题建议：**
`From MuJoCo to Policy: A Colab-Friendly Go2 Locomotion Pipeline`

**副标题建议：**
`Embodied AI Homework Tutorial`

**这一页要做的事：**
- 告诉学生今天的目标不是公式推导
- 告诉学生今天要看完整 pipeline

**讲者备注：**
保持极简，不要在第一页塞太多内容。

---

## Slide 2：What you will be able to do after this tutorial

**建议内容：**
- read the task definition
- run baseline training
- restore a checkpoint
- render a demo
- run the public benchmark
- know where to modify the homework

**讲者备注：**
这一页相当于 learning outcomes，帮助学生建立预期。

---

## Slide 3：The full pipeline in one picture

**建议主图：**
`Go2 XML -> Playground Env -> PPO -> Checkpoint -> Demo Rollout -> Public Benchmark -> Real-Robot Shortlist`

**讲者备注：**
这页非常关键。  
整场 tutorial 后面所有内容都应该不断回到这一张图。

---

## Slide 4：Official notebook vs our course code

**建议用左右对比结构：**

左边：Official MuJoCo Playground locomotion notebook
- built-in environments
- tutorial style
- direct in-notebook training
- broad Playground introduction

右边：Our course version
- local Go2 environment
- assignment structure
- explicit train / test / benchmark scripts
- Colab-friendly training budget
- submission-oriented artifacts

**讲者备注：**
要明确说：
我们不是重复官方 notebook，而是在它之上做了“课程化重构”。

---

## Slide 5：Software stack

**建议主图：**
四层堆栈图

- MuJoCo XML / robot model
- MuJoCo Playground environment
- Brax PPO trainer
- course scripts and benchmark

**讲者备注：**
不要把所有库都展开。  
重点讲“谁负责什么”。

---

## Slide 6：Repository map

**建议内容：**
用文件树或模块图展示：

- `go2_pg_env/joystick.py`
- `go2_pg_env/randomize.py`
- `train.py`
- `test_policy.py`
- `generate_public_rollout.py`
- `public_eval.py`
- `course_config.json`

**讲者备注：**
这一页的目标是让学生知道：
以后查问题应该去哪里看。

---

## Slide 7：Observation, action, and privileged information

**建议布局：**
左侧讲 actor observation  
右侧讲 critic privileged observation  
底部讲 action

**要点：**
- actor uses deployable observations
- critic sees more simulator-only signals during training
- policy action is joint target offset, not raw torque

**讲者备注：**
这页最好配一个 very simple data-flow diagram。

---

## Slide 8：Reward design

**建议不要列满所有 reward 项。**

按三类展示：

### Task
- linear velocity tracking
- yaw tracking

### Stability
- orientation
- vertical velocity
- body angular velocity
- termination

### Smoothness / gait
- action rate
- energy
- foot slip
- foot air time

**讲者备注：**
这一页要讲“reward 设计思想”，而不是“reward 术语大全”。

---

## Slide 9：Domain randomization and sim-to-real

**建议内容：**
- friction
- mass
- armature
- COM shift
- reset noise

**建议加一句大字：**
`Train on a family of simulators, not on a single perfect simulator.`

**讲者备注：**
这句话很适合作为学生记忆点。

---

## Slide 10：Two-stage curriculum

**建议用两段式流程图：**

### Stage 1
- smaller command range
- easier task
- learn stable walking first

### Stage 2
- larger command range
- stronger action-rate / energy regularization
- refine robustness and smoothness

**讲者备注：**
这一页解释为什么 baseline 不是单阶段直接猛训。

---

## Slide 11：What each script does

**建议做成四块职责图：**
- `train.py`
- `test_policy.py`
- `generate_public_rollout.py`
- `public_eval.py`

**每块只保留 2 - 3 行职责说明。**

**讲者备注：**
这是学生最容易建立“工程层面掌握感”的一页。

---

## Slide 12：Live demo plan

**建议列出你现场会跑的命令：**
- `python inspect_env.py --stage-name stage_2`
- `python test_policy.py ...`
- `python generate_public_rollout.py ...`
- `python public_eval.py ...`

**讲者备注：**
提前告诉学生等下 live demo 看什么，可以显著降低他们的迷失感。

---

## Slide 13：What students are allowed to change

**建议分两列：**

### You may change
- reward design
- command ranges
- observation choices
- domain randomization
- curriculum settings

### Please do not change
- public evaluator
- rollout file format
- checkpoint restore logic

**讲者备注：**
这一页是避免之后作业跑偏的关键页。

---

## Slide 14：Common failure modes

**建议内容：**
- only looking at the last checkpoint
- confusing reward with benchmark
- changing too many things at once
- not checking a deterministic rollout before submission
- focusing on notebook execution but not understanding the file structure

**讲者备注：**
建议配一个 “debug checklist” 小框。

---

## Slide 15：Take-home message

**建议只保留 3 句：**
1. A robot policy is a pipeline, not just a neural network.
2. Benchmarking is different from training reward.
3. If you understand the files, you understand the system.

**讲者备注：**
最后一页尽量短，给学生一个稳定收尾。

---

# 现场演示建议

## 最稳妥的演示顺序
1. `inspect_env.py`
2. 打开 `joystick.py` 展示 obs / action
3. 打开 `course_config.json` 展示 stage_1 / stage_2
4. `test_policy.py` 恢复 checkpoint 并输出 demo
5. `generate_public_rollout.py`
6. `public_eval.py`
7. 打开结果 JSON

## 不建议现场演示的内容
- full training from scratch
- 太长的 reward 数学展开
- 太细的 PPO 推导
- 太多 JAX 内部细节

因为这会破坏这次 tutorial 的主目标：  
**让学生掌握 pipeline，而不是淹没在实现细节里。**
