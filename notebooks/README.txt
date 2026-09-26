Data Sheet S1 — notebooks for:
"A Quantitative Fall Threshold for Graded Unilateral Motor-Command Attenuation in a Simulated
Feedforward Humanoid Locomotion Policy" (revised version)

Contents
1. g1_training_cross_seed.ipynb
   Training of the eight Unitree G1 locomotion policies (mjlab 1.2.0, PPO; learning-algorithm
   seeds 1-8, environment seed 100, 3,000 iterations, 2,048 environments). Contains the exact
   training command line. Outputs have been cleared.
2. g1_hemiparesis_threshold.ipynb
   Original analysis notebook: unilateral attenuation, coarse sweep, bisection (interval
   corrected to alpha = 0.6-0.8 at revision), fine sweep, speed dependence. Section 9 is kept
   for transparency only; its joint-angle measurement is superseded (see the revision note in
   the notebook).
3. g1_hemiparesis_revision.ipynb
   Revision analyses: action-pipeline check (A), specification record (A2), demonstration of
   the reset artifact (B), corrected fall-onset kinematics (C), repeated trials under 10
   randomized conditions (D, optional D2), statistics (E, E2), figures (F), checkpoint export
   (G), and re-measurement of the coarse and speed-dependence sweeps (H).

Notes
- Re-running requires a CUDA GPU and the trained checkpoints (logs/rsl_rl/g1_velocity/*_seed{1..8}),
  which are publicly available together with the per-trial result tables (see the Data
  Availability statement of the article).
- The GPU-accelerated simulator (MuJoCo Warp) is not bitwise deterministic across runs;
  individual trials exactly at the fall threshold may change outcome between runs.
