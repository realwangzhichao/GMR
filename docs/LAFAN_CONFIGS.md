# X2 Ultra / T2 的 LAFAN1 映射

当前默认使用简单参数，走原生 GMR IK 和模型硬限位；G1 映射保持原样。

| 参数 | X2 Ultra | T2 |
|---|---:|---:|
| 身体、腿、脚比例 | 0.73 | 0.88 |
| 双臂统一比例 | 0.81 | 0.84 |
| 脚局部位置偏移 xyz（m） | [-0.03, 0, 0.03] | [-0.01, 0, -0.02] |
| 脚位置/朝向 cost，两阶段相同 | 150 / 20 | 150 / 20 |

旋转使用 `0`、`±0.5`、`±0.70710678` 等轴转换常数。
后续调参直接修改 `general_motion_retargeting/ik_configs/bvh_lafan1_to_x2.json`
或 `bvh_lafan1_to_t2.json`，无需重新运行自动标定。

## 导出与预览

直接使用 GMR 原生批量入口（在 GMR 目录执行）：

```bash
conda activate gmr
python scripts/bvh_to_robot_dataset.py \
  --src_folder ../human_data/lafan1 \
  --tgt_folder ../human_data/lafan1_morphmimic/my_batch/pkl/x2/lafan1 \
  --robot agibot_x2_ultra
```

T2 改为 `--robot booster_t2`，G1 为 `--robot unitree_g1`，各自使用独立输出目录。
脚本递归处理源目录中的 BVH，保存完整帧，使用 CUDA 计算局部身体位置；没有额外的
首帧重复求解。只想导出部分动作时，先将所需 BVH 放入单独的源目录。

```bash
python scripts/vis_robot_motion.py --robot agibot_x2_ultra \
  --robot_motion_path ../human_data/lafan1_morphmimic/my_batch/pkl/x2/lafan1/walk1_subject1.pkl
```

单段实时预览也可使用 `scripts/bvh_to_robot.py --bvh_file FILE --robot NAME`。
转为 MorphMimic 的 50 Hz 数据时，用其 `convert_gmr --source-xml` 指定实际源模型：

| 机器人 | GMR 模型路径 |
|---|---|
| G1 | `assets/unitree_g1/g1_mocap_29dof.xml` |
| X2 | `assets/agibot_x2_ultra/x2_ultra.xml` |
| T2 | `assets/booster_t2/t2.xml` |

转换器据此读取关节顺序，原生 PKL 无需额外添加关节名称或时间戳。

## 已完成的同口径检查

2026-11-10：共同使用走路、跑步、舞蹈三个完整片段，每个机器人 18,920 帧、30 Hz。
这些历史诊断在首帧重复求解 30 次后开始保存；上面的原生批量入口没有这一步，因此
表格不是原生批量入口零预热输出的逐帧复测结果。
下表的支撑片段来自相同源动作的低脚速/低高度判定，不是真实接触标签；测量的是
每个模型自身足底碰撞几何的高度。

| 配置 | 支撑段 ±1 cm 内 ↑ | 高于 1 cm ↓ | 低于 −1 cm ↓ | 关节单帧变化 P99（rad） |
|---|---:|---:|---:|---:|
| 原生 G1 | 9.74% | 89.94% | 0.33% | 0.262 |
| 原生 H2 | 35.11% | 4.00% | 60.89% | 0.263 |
| 原生 T1 | 78.67% | 19.40% | 1.93% | 0.265 |
| 当前 X2 | 78.02% | 20.94% | 1.04% | 0.251 |
| 当前 T2 | 77.65% | 21.15% | 1.20% | 0.268 |

X2/T2 在这些指标上与原生 T1 接近，全部输出有限且没有明显硬限位越界。
T2 跑步仍存在较大单帧关节变化峰值（1.42 rad）。相比此前三位小数版本，
当前简单参数减少穿地但增加悬空。这些是运动学检查，尚未验证策略动态跟踪。

## 诊断记录

临时标定、诊断、渲染、额外导出脚本及对应测试已移出 GMR。完整评测数据和修改前说明保留在：

- [五机器人对照结果](../../human_data/lafan1_morphmimic/native_mapping_comparison_20261110/comparison.json)
- [诊断脚本与完整原说明归档](../../human_data/lafan1_morphmimic/gmr_diagnostics_archive_20261110/docs/LAFAN_CONFIGS.md)
- [清理文件清单](../../human_data/lafan1_morphmimic/gmr_diagnostics_archive_20261110/cleanup_manifest.json)
