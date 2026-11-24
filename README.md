# Data for "Active Learning with Bayesian Multi-Fidelity Laplace Neural Operators for Oscillatory Parametric PDEs"

This repository holds the datasets and numerical results for the paper
*Active Learning with Bayesian Multi-Fidelity Laplace Neural Operators for Oscillatory Parametric PDEs*
(B. Kim, H. Zheng, M. Penwarden, G. Lin), submitted to *Machine Learning: Science and Technology*.

The paper has three numerical experiments:

| ID | Problem | Parameters (2D) | LF data | HF data |
|---|---|---|---|---|
| Ex1 | Forced Lorenz system | forcing amplitude `a` in [0.01, 1.0], `rho` in [5.0, 8.0] | 512 time points, t in [0, 20] | 2048 time points, t in [0, 20] |
| Ex2 | Forced Duffing oscillator | forcing amplitude `C` in [6.0, 8.0], damping `c` in [0.4, 0.5] | 1024 time points, t in [0, 20] | 2048 time points, t in [0, 20] |
| Ex3 | Composite cantilever beam | `a1` in [0.7, 1.3], `xi2` in [8000, 12000] | 1D Euler-Bernoulli beam, 101 time steps | 2D plane-stress perforated beam, 101 time steps |

## Files

Repository: <https://github.com/Kim4853/MF-LNO-data>

GitHub limits single files to 100 MB, so the archives are attached to the [Releases](https://github.com/Kim4853/MF-LNO-data/releases) page of this repository, not stored in the git tree.

| Archive | Size | Contents |
|---|---|---|
| `MF_LNO_datasets.zip` | ~141 MB | Raw LF/HF training, validation and test data for all three experiments |
| `MF_LNO_results.zip` | ~308 MB | Final results: prediction snapshots, run summaries, error tables, figures, training logs, and the figures in the paper |
| `MF_LNO_artifacts_ex1_lorenz.zip` | ~1.3 GB | Per-stage active-learning bundles for Ex1 (all seeds, all HF budgets) |
| `MF_LNO_artifacts_ex2_duffing.zip` | ~2.0 GB | Per-stage active-learning bundles for Ex2 |
| `MF_LNO_artifacts_ex3_beam.zip` | ~158 MB | Per-stage active-learning bundles for Ex3 |
| `SHA256SUMS.txt` | | Checksums for all archives (`sha256sum -c SHA256SUMS.txt`) |

To reproduce the reported numbers and figures, you only need `MF_LNO_datasets.zip` and `MF_LNO_results.zip`. The artifact archives are large and hold every intermediate active-learning stage.

---

## 1. `MF_LNO_datasets.zip`

```
MF_LNO_datasets/
├── ex1_lorenz/
│   ├── lorenz_2048.mat            # HF resolution
│   ├── lorenz_512.mat             # LF resolution
│   └── lorenz_cd_metadata.npz     # parameter values and split definition
├── ex2_duffing/
│   ├── duffing_2048.mat           # HF resolution
│   ├── duffing_1024.mat           # LF resolution
│   └── duffing_Cc_metadata.npz
└── ex3_beam/
    ├── mf_lno_beam_dataset_a1_xi2_20x20.npz   # HF (2D) and LF fields + tip trajectories
    ├── lf_beam_dataset_a1_xi2_20x20.npz       # LF (1D Euler-Bernoulli) dataset
    └── beam_with_holes.msh                    # finite element mesh of the HF beam
```

### Ex1 / Ex2 (`.mat`, MATLAB v5; read with `scipy.io.loadmat`)

| Key | Shape | Meaning |
|---|---|---|
| `T` | (N_t, 1) | time grid on [0, 20] (N_t = 2048 HF; 512 Lorenz LF; 1024 Duffing LF) |
| `f_train`, `u_train` | (100, N_t) | input forcing / output trajectory, training candidate set |
| `f_vali`, `u_vali` | (15, N_t) | validation set |
| `f_test`, `u_test` | (100, N_t) | test set |

**Splits** (from `*_metadata.npz`):
- The training candidate set is a 10 × 10 uniform grid over the 2D parameter domain (100 instances). HF training samples are chosen from this set.
- The validation set (15) and the test set (100) are drawn at random from the same parameter domain (`split_mode = full_grid_train_plus_random_eval`, `seed = 0`).
- The metadata files store the parameter values of every split (`train_*`, `vali_*`, `test_*`), the parameter ranges, `low_res` / `high_res`, and the time interval.

**Important: the LF `.mat` files hold *uncorrupted* solutions on the coarse grid.** The LF discrepancy described in the paper is applied when the data are loaded in the training code:
- Lorenz: `u_L(t) = u_H(t)(1 + 0.3 sin t) + 1.5 + 0.0225 t`
- Duffing: the parameter-dependent low-pass / amplitude map defined in the paper

The corrupted LF outputs used in training are saved as `y_low_corrupted` in the artifact bundles (Section 3).

### Ex3 (`.npz`; read with `numpy.load`)

`mf_lno_beam_dataset_a1_xi2_20x20.npz`:

| Key | Shape | Meaning |
|---|---|---|
| `params` | (400, 2) | (`a1`, `xi2`) on a 20 × 20 grid |
| `a1_values`, `xi2_values` | (20,) | grid values |
| `t` | (101,) | time grid on [0, 20] |
| `x` | (128,) | axial coordinate on [0, 9.8] |
| `Y_H`, `Y_L` | (400, 101, 128) | HF and LF displacement fields |
| `delta` | (400, 101, 128) | `Y_H - Y_L` |
| `tip_H`, `tip_L` | (400, 101) | HF and LF tip-displacement trajectories (the operator-learning targets) |
| `fixed_a2`, `fixed_tau1`, `fixed_tau2`, `fixed_sigma1`, `fixed_sigma2` | scalar | fixed load parameters |

`lf_beam_dataset_a1_xi2_20x20.npz` contains the LF beam model on its native 25-node grid (`x` (25,), `W_L` (400, 101, 25), `tip_L` (400, 101)) together with the same parameters.

---

## 2. `MF_LNO_results.zip`

```
MF_LNO_results/
├── ex1_lorenz/   {exports, figures, train_logs}
├── ex2_duffing/  {exports, figures, train_logs}
├── ex3_beam/     {exports, figures, train_logs}
└── paper_figures/   # exact figure files included in the manuscript
```

### `exports/`

Model labels in file names:

| Label | Method in the paper |
|---|---|
| `lno_al` | MF-LNO (Active), the proposed uncertainty-guided acquisition |
| `lno_random` | MF-LNO (Random) |
| `deeponet_random` | MF-DeepONet (Random) |

Tags in file names:
- `conservative` / `finaluq`: names of the two active-learning run configurations used in the training notebooks.
- `seed0` … `seed4`: the five random seeds.

| File pattern | Contents |
|---|---|
| `run_summary_<model>_test100[_tag]_seed<k>.npz` | Error versus number of HF trajectories: `mf_num_hf`, `mf_avg_rel_l2`, `mf_max_rel_l2` (test set), selected HF indices, LF/HF loss histories. These are the data behind the error-vs-HF-budget figures and tables. |
| `low_stage_snapshot_<model>_seed<k>.npz` | Phase 1 (LF model) predictions and losses |
| `high_stage_hf<N>_snapshot_<model>_<tag>_seed<k>.npz` | Phase 2 (MF model) state at N HF trajectories: test inputs/targets (`x_test_*`, `y_test_high`), predictions (`pred_test_high_final`), posterior mean/std (`uq_pred_mean`, `uq_pred_std`), interval bounds (`uq_ci_low`, `uq_ci_high`), selected HF indices, loss histories, learned `alpha` |
| `final_trajectory_error_table.tex` | LaTeX source of the representative-trajectory error tables |

### `figures/`

These are the post-processing figure outputs for each experiment. The figures in the manuscript are in `paper_figures/`.

### `train_logs/`

Training and validation loss histories per run (`.npz`, timestamped), organised by model.

### `paper_figures/`

| Folder | Paper figures |
|---|---|
| `1d_lorenz/` | Ex1: acquisition maps, representative predictions, error vs. HF budget, reSGLD sensitivity |
| `1d_duffing/` | Ex2: acquisition maps, representative predictions, error vs. HF budget, learning-rate ablation |
| `2d_beam/`, `beam/` | Ex3: configuration, mesh, LF/HF trajectories, acquisition maps, predictions, error vs. HF budget |
| `flowchart.pdf`, `reSGLD.pdf` | Method schematics |

---

## 3. `MF_LNO_artifacts_<example>.zip`

Each archive holds every intermediate active-learning stage for every seed:

```
MF_LNO_artifacts_<example>/artifacts/<model>_test100[_tag]_seed<k>_<stage>_hf<N>_{train,test}_bundle.npz
```

- `<stage>` is `before_al`, `stage_hf<N>` or `after_al`.
- `train` and `test` are evaluated on the training candidate set and on the test set, respectively.

Typical keys:

| Key | Meaning |
|---|---|
| `selected_high_indices` | indices of the HF trajectories used at this stage |
| `grid_low`, `grid_high` | LF and HF time grids |
| `x_low`, `x_high` | LF and HF inputs (forcing and parameter channels) |
| `y_low_clean`, `y_low_corrupted` | LF outputs before and after the LF discrepancy map |
| `y_high_truth` | HF reference solution |
| `mf_final_pred` | MF prediction |
| `mf_posterior_mean`, `mf_posterior_std` | reSGLD posterior predictive mean and standard deviation |
| `mf_rel_l2` | relative L2 error per sample |
| `posterior_sample_count` | number of posterior samples |

---

## Loading example

```python
import numpy as np
from scipy.io import loadmat

hf = loadmat("MF_LNO_datasets/ex1_lorenz/lorenz_2048.mat")
print(hf["u_test"].shape)          # (100, 2048)

summ = np.load("MF_LNO_results/ex1_lorenz/exports/run_summary_lno_al_test100_conservative_seed0.npz")
print(summ["mf_num_hf"], summ["mf_avg_rel_l2"])
```

## Code

The code (Laplace neural operator, reSGLD optimizer, training and post-processing notebooks) is not part of this data repository. See the Code Availability statement of the paper.

## Citation

If you use these data, please cite the paper (citation details will be added after publication).

## License

The data in this repository are licensed under the
[Creative Commons Attribution-NonCommercial 4.0 International License (CC BY-NC 4.0)](https://creativecommons.org/licenses/by-nc/4.0/).
See [`LICENSE`](LICENSE) for the full text.

- You may share and adapt the data for **non-commercial** purposes, with appropriate credit (cite the paper).
- Commercial use requires prior written permission from the authors (contact: guanglin@purdue.edu).
- The data are provided "as is", without warranty of any kind.
