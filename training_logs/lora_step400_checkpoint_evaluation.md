# Radeon Cloud LoRA Step 400 测试记录

**日期：** 2026-09-27
**目的：** 记录 500 步 LoRA 训练、checkpoint 恢复点、Step 400 合并，以及与官方 RoboTwin checkpoint 的同设置闭环评测。

## 环境与评测设置

- Radeon Cloud：8 张 Radeon PRO W7900D；PyTorch 2.9.1、ROCm 7.2.1。
- 评测任务：`adjust_bottle`，配置 `demo_clean`，`seed=0`，10 episodes。
- 评测参数：`expert_check=true`、`accept_expert_info_on_failure=true`、`eval_batch=false`、`eval_video_log=false`。
- 运动规划环境：ROCm + MPLib，关闭 CuRobo；两组评测使用相同参数和环境，只切换模型 checkpoint。

## LoRA 训练

- 8 GPU LoRA 训练到达 `Step 500/500`；约 27 分 52 秒训练循环时间，整体约 28 分 11 秒。
- 解析到 499 条逐步指标（step 2 至 500）。Loss 均值为 0.25213；前 50 步均值 0.35414，最后 50 步均值 0.19112，下降约 46%；step 500 Loss 为 0.1591，VLA Loss 为 0.1581。
- 平均 step time 为 2.9485 秒。训练日志中未见 NaN 或训练阶段 OOM。
- 训练结束时写入 `global_step_500` 因 `/workspace` 空间不足失败（`OSError: [Errno 28] No space left on device`）。`global_step_100/200/300/400` 的模型与优化器 checkpoint 完整；`global_step_500` 是不完整副本，不可作为恢复点。

训练日志副本：`training_logs/lora_500steps_8gpu.log`。

## 存储迁移与合并

- 原训练输出约 107.4 GB。复制至实例临时盘 `/root/robotwin-scratch/outputs/lora_500steps_8gpu` 后执行内容校验，源与副本无差异；随后从共享 `/workspace` 移除已迁移的 checkpoint、模型资产和训练配置，只保留日志与 TensorBoard event 文件。
- 合并输入：`checkpoints/global_step_400`；使用 `merge_lora_dcp.py`、rank 8、alpha 16。成功合并 144 个 LoRA 层，产物约 12 GB；索引文件引用的 3 个权重分片均存在且非空。
- 合并模型：`/root/robotwin-scratch/outputs/lora_500steps_8gpu/merged_checkpoint/global_step_400/hf_ckpt`。
- 临时盘是实例 overlay，销毁或替换实例可能丢失其中的原始 checkpoint 与合并模型。正式保留前应另行下载或备份。

## 闭环评测对照

| 模型 | Checkpoint | 成功数 | 成功率 |
|---|---|---:|---:|
| 本次微调 LoRA | `global_step_400`（DCP 合并后评测） | 0/10 | 0% |
| 官方 RoboTwin 模型 | `global_step_50000/hf_ckpt` | 5/10 | 50% |

官方模型位于只读挂载：`/models/robotwin-persistent/models/robbyant_lingbot-vla-v2-6b-robotwin/checkpoints/global_step_50000/hf_ckpt`；评测配置为同目录上级的 `lingbotvla_cli.yaml`。已确认官方模型索引和 6 个权重分片完整。

两次评测均出现 `missing pytorch3d` 提示和 IK 规划失败日志；但官方模型仍成功 5 集，因此这些提示本身不能解释 LoRA 的 0/10。结果显示该 LoRA checkpoint 在本次任务上明显弱于官方 RoboTwin checkpoint；仅凭 10 集、单一任务不能推断所有任务表现，也不能仅凭训练 Loss 判断任务成功率。

## 结果与运行日志

云实例持久 `/workspace/runtime/outputs/logs/` 中保留：

- `merge_lora_step400.log`
- `merged_server_step400.log`
- `lora_step400_eval.log`、`lora_step400_eval_result.txt`
- `official_robotwin_ckpt_server_compare.log`
- `official_robotwin_ckpt_eval_compare.log`、`official_robotwin_ckpt_eval_compare_result.txt`

两次评测与模型服务均已结束；长 benchmark 未运行。上述云端日志与模型 checkpoint 的生命周期不同：日志在持久卷，训练与合并模型在临时盘。