# V7 — Final Submission

This folder contains my submission for the EEC289A omnidirectional Go2 locomotion homework.

## TL;DR

V7 (single-stage training mirroring official mujoco_playground Go1) successfully tracks
all 6 directional commands (+vx, -vx, +vy, -vy, +yaw, -yaw) plus combined commands.

| Version | Strategy | Public composite | Backward tracking |
|---|---|---|---|
| V2 (baseline) | Forward-only stage_1 | (failed demo) | ❌ 0% |
| V3 | Bug fix + relaxed reward penalties | 0.974 | ❌ 1% |
| V6 | Symmetric vx+vy stage_1 | 0.965 | ❌ 1% |
| **V7** | **Single-stage, mirrors official mujoco_playground** | **0.966** | **✅ 78–91%** |

## Contents

- `configs/`           — Training configs for V3 / V6 / V7
- `figures/`           — 4 analysis figures from custom evaluation
- `best_checkpoint/`   — V7 final policy weights
- `results/`
  - `demo_v7.mp4`             — 70-second qualitative demo
  - `public_eval_*.json`      — Standardized benchmark results
  - `training_summary_v7.json` — Training log
  - `demo_summary_v7.json`     — Demo metrics

See ../short_report.pdf for full report (5 pages).

## Reproducing V7

```bash
# Apply V7 config
cp my_work/configs/course_config_v7.json configs/course_config.json

# Train (30M steps, ~13 minutes on A100)
python train.py \
  --config configs/colab_runtime_config.json \
  --stage both \
  --stage1-steps 15000000 \
  --stage2-steps 15000000 \
  --output-dir artifacts/run_v7
```
