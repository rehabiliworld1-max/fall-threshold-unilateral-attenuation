# Fall threshold for unilateral motor-command attenuation in a simulated humanoid locomotion policy

Supporting materials (code, per-trial results and trained policy checkpoints) for:

> Obaru, K. *A Quantitative Fall Threshold for Graded Unilateral Motor-Command Attenuation in a Simulated Feedforward Humanoid Locomotion Policy.* Frontiers in Robotics and AI (under review; manuscript no. 10524680).

All experiments were performed entirely in simulation (MuJoCo via mjlab) with a simulation model of the Unitree G1 humanoid. No physical robot was used.

## Overview

Eight feedforward locomotion policies were trained with PPO for the Unitree G1 model. They differ only in the learning-algorithm seed. For each policy, the output-layer weights and biases of all 13 actuated joints on one side of the body were multiplied by a common factor α (the *remaining motor output*), without any retraining. This scales the commanded deviation from the default posture on that side by exactly α. It leaves torque limits and joint stiffness unchanged. A fall is defined as trunk tilt exceeding 70° within 250 control steps (5 s at 50 Hz).

The repository contains everything needed to inspect or reproduce the reported fall thresholds, the left–right comparisons, the speed dependence, the repeated-trial analysis and the fall-onset kinematics.

## Repository structure

```
notebooks/
  g1_training_cross_seed.ipynb     Training of the eight policies (seeds 1-8)
  g1_hemiparesis_threshold.ipynb   Original analysis: sweeps, bisection, speed dependence
  g1_hemiparesis_revision.ipynb    Revision analyses (sections A-H, see below)
  README.txt
results/
  H_coarse.csv                     Coarse sweep (Table 1)
  H_speed.csv                      Fine sweep at four speeds (Tables 2-3, Figure 2, Table S1)
  D_variability.csv                Repeated trials, 10 conditions per policy (Figure 1, Table S2)
  C_mechanism_summary.csv          Fall-onset kinematics, one row per trial (Section 3.6)
  C_trajectories/*.npz             Per-step trajectories behind Figure 3
  H_table1_coarse.csv, H_table2_fine.csv, H_table3_speed.csv, E2_side_by_speed.csv
                                   Summary tables computed from the files above
  A2_joint_table.csv, A2_spec.json, A_action_pipeline.csv
                                   Model and software specification (Table S3, Material S4)
  G_checkpoints.csv                Checkpoints used for each seed
  figures/                         Figures 1-3 (300 dpi PNG and TIFF)
checkpoints/
  g1_velocity_seed{1..8}_model_2999.pt   Final trained policy for each seed
```

## Result files

The condition labels in the data files are `right_half_paresis` and `left_half_paresis`. They correspond to *right-side attenuation* and *left-side attenuation* in the article. `level` is the remaining motor output α (1.0 = unmodified, 0.0 = complete ablation).

| File | Columns | Content |
| --- | --- | --- |
| `H_coarse.csv`, `H_speed.csv` | condition, seed, level, speed_mps, fell_over, fall_step | One single-rollout trial per row (environment seed 100). `fall_step` is the control step of fall onset (empty if no fall). |
| `D_variability.csv` | condition, seed, level, ic, fell_over, fall_step | Repeated trials at 0.5 m/s. `ic` (0-9) indexes the randomized condition: start position, drop height, foot friction, encoder bias and torso centre-of-mass offset. |
| `C_mechanism_summary.csv` | condition, seed, level, fell_over, fall_step, `q_*`, `tgt_*` | Mean absolute deviation (deg) from the default posture of the actual joint angles (`q_`) and commanded targets (`tgt_`). Values are given for attenuated (`paretic`) and unattenuated (`nonparetic`) joints, at fall onset (`_at_fall`), over the preceding 0.5 s (`_pre05`) and during steady walking at α = 1.0 (`_walk`). |
| `C_trajectories/*.npz` | `q`, `target`, `tilt`, `height`, `q0` | Per-control-step joint angles, commanded targets (rad), trunk tilt (rad) and base height (m) up to fall onset, plus the default posture. File names give condition, seed and level (e.g. `L65` = α 0.65). |

## Checkpoints

`checkpoints/` holds the final checkpoint (iteration 2999) of each of the eight policies, 5.3 MB each. To use them with the notebooks, rename and place each file as `logs/rsl_rl/g1_velocity/<any_name>_seed<N>/model_2999.pt`. The notebooks select the most recent run directory whose name ends in `_seed<N>`.

## Reproducing the analyses

Requirements: a CUDA GPU. The results were produced on an NVIDIA Tesla T4 in Google Colab.

| Package | Version |
| --- | --- |
| mjlab | 1.2.0 |
| MuJoCo / MuJoCo Warp | 3.5.0 / 3.6.0 |
| warp-lang | 1.12.1 |
| rsl-rl-lib | 5.0.1 |
| PyTorch | 2.11.0 for evaluation (training version not recorded; mjlab 1.2.0 requires ≥ 2.7) |

Open `notebooks/g1_hemiparesis_revision.ipynb` in Colab with a GPU runtime and run the cells from the top. The sections are:

- **A, A2**: Action-generation pipeline check and specification record.
- **B**: Demonstration that the originally reported convergence to the default posture was an artifact of reading the state after the environment's automatic reset.
- **C**: Corrected fall-onset kinematics.
- **D (D2 optional)**: Repeated trials under 10 randomized conditions.
- **E, E2**: Statistics (sign test, Wilcoxon signed-rank test, bootstrap confidence intervals, variance decomposition).
- **F**: Figures.
- **G**: Checkpoint export.
- **H**: Re-measurement of the coarse and speed-dependence sweeps.

The longer sections (D, D2 and H) take from about ten minutes to about an hour each on a T4, and all of them can resume if interrupted.

**Note on determinism.** The GPU-accelerated simulator (MuJoCo Warp) is not bitwise deterministic across runs. Individual trials lying exactly at the fall threshold can change outcome between repeated evaluations of an identical condition, for example left-side attenuation at α = 0.60. Re-running the notebooks may therefore change individual near-threshold cells by one policy. Group-level thresholds and the repeated-trial statistics are robust to this.

## Citation

If you use these materials, please cite the article above. A full citation will be added here on publication.

## License

See `LICENSE`.

## Contact

Kenshi Obaru, rehabiliworld (Kumamoto City, Japan). research@rehabiliworld.com
