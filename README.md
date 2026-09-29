# GuiderQ SC'26 Artifact

Welcome to the GuiderQ SC'26 Artifact Evaluation repository for paper 515. This README describes the prepared artifact.

## Artifact information

| Field | Value |
| --- | --- |
| Image name | `sc26ae-515-latest-ae-20260728-v3` |
| Image UUID | `aa377dcf-9d95-4a3f-8689-4dc4ffec544a` |
| Chameleon site | `CHI@UC` |
| Base system | Ubuntu 22.04 CUDA, x86-64 |
| Disk format | QCOW2 |
| Artifact root | `/home/cc/guiderq/latest_ae` |
| Frozen Python environment | `/home/cc/quartz-ae/env` |
| Python executable | `/home/cc/quartz-ae/env/bin/python` |

The prepared image includes the software environment, source code, locked checkpoints and Guider models, ECC rule sets, benchmark inputs, preprocessing materials, evaluation scripts, and reference results. No environment reconstruction or compilation is required in the prepared image.

The image and parallel AE workflow were validated on a four-GPU NVIDIA A100 80 GB PCIe node. The documented hardware options are x86 nodes with NVIDIA A100 PCIe or A100 NVLink GPUs. The commands below assume that the prepared environment is already available.

### Software

The artifact uses Python 3.10 on Linux (tested with kernel 5.15.0) and CUDA >= 11.7. PyTorch and DGL were built against CUDA 11.7; the prepared system runtime is CUDA 12.8.

| Package | Version |
| --- | --- |
| PyTorch | 2.0.1 |
| DGL | 0.9.1post1 |
| Quartz | 0.1.0 |
| NumPy | 1.26.4 |
| Matplotlib | 3.7.5 |
| Qiskit | 2.1.1 |
| Pandas | 2.3.3 |

Environment manifests are included under `latest_ae/environment/` for auditing and fallback reconstruction.

### Inputs and preprocessing

The artifact bundles 24 Quartz-suite and 38 MQTBench benchmark circuits, representing 62 unique circuit names. The bundled representations include:

- The 24 Quartz-suite high-level sources and their TDG inputs under `latest_ae/circuits/`.
- Six checkpoint-compatible, paper-preprocessed IBM inputs and six Nam inputs, together with preprocessing records and tools.
- All 38 MQTBench circuits from Table V under `latest_ae/circuits/mqtbench_table_v/`, including source, Qiskit-O3-with-measurement, and Nam PPO-ready representations.

For efficiency, the selected circuits used in the AE evaluation were **preprocessed in advance before the optimization stage**. The corresponding preprocessing files and outputs are included under `latest_ae/`; preprocessing programs are under `latest_ae/preprocessing/`, and circuit inputs are under `latest_ae/circuits/`. The locked IBM, Nam, and TDG ECC rule sets are under `latest_ae/ecc/`.

**GuiderQ and Quarl use the same preprocessed inputs in the AE workflow.** These inputs may have fewer gates than the original declared circuit sizes reported in the paper. AE optimizer gate-count reductions are measured relative to the preprocessed input supplied to both optimizers, rather than the original declared circuit size. For example, the 27,880 declared gates for `realamprandom_indep_130` describe the original circuit, not necessarily the optimizer's input gate count.

### Reduced fixed-seed AE scope

The AE workflow uses pre-trained checkpoints and locked Guider models in a **resumable, fixed-seed 28-task matrix**. It covers:

- **Table IV:** the selected 10 TDG circuits from the Quartz suite.
- **Figures 6 and 8:** the ablation experiments included in the reduced AE matrix.
- **Table V:** a representative long Nam circuit, `realamprandom_indep_130`, with paired GuiderQ/Quarl RL-OAC tasks. Each task has a budget of **2 hours 52 minutes**, or approximately **5.73 aggregate GPU-hours** for the pair.

This reduced protocol does not rerun all circuits in Tables IV and V, all five seeds, or the full pretraining procedure. Its results validate the representative AE subset and should not be interpreted as reproducing the full aggregate paper evaluation.

The **estimated execution budget is approximately 29.40 aggregate GPU-hours** for the reduced reproduction matrix. This corresponds to an ideal four-GPU wall time of approximately 7.35 hours and a budget-aware queue estimate of approximately 7 hours 32 minutes. Allow up to eight hours for launch overhead, validation, and queue imbalance. These are planning estimates, not measured runtimes or guarantees; actual runtime may vary with execution overhead, system conditions, and GPU scheduling. Smoke-test estimates are listed separately below.

## Artifact Evaluation commands

Unless otherwise stated, run all commands from `/home/cc/guiderq/latest_ae`. Relative result paths below are rooted in that directory.

### 1. Set up and validate the artifact

Estimated time: under one minute; 0 GPU-hours.

```bash
cd /home/cc/guiderq/latest_ae
export LATEST_AE_PYTHON=/home/cc/quartz-ae/env/bin/python
export AE_GPUS="0 1 2 3"

./ae.sh check
```

Confirm successful checks for Python, CUDA, manifests, checksums, checkpoints, benchmark inputs, bundled Quartz paths, and locked ECC/actor action dimensions. The expected action dimensions are TDG = 3,551, IBM = 2,651, and Nam = 28,317.

### 2. Run all Functional/Reusable smoke tests

Estimated time: up to 10 minutes of wall time on four A100 GPUs; up to 0.67 GPU-hours.

```bash
./ae.sh smoke-all
```

This is the recommended end-to-end functional check for all three contributions: compact global representation (C1), guided exploration (C2), and scalable segment optimization (C3). It uses four concurrent GPU lanes to cover the TDG, IBM, and Nam paths and writes verified logs, final checkpoints, and QASM outputs under:

```text
results/history/smoke-runs/full_parallel_<timestamp>/
```

### 3. Test individual components (optional)

The following commands use one GPU by default and process the TDG, IBM, and Nam gate sets sequentially. The combined estimated budget is approximately 0.25 GPU-hours: approximately 0.05 for C1, 0.05 for C2, and 0.15 for C3.

```bash
BUDGET_SEC=60 ./ae.sh smoke-multihead all
BUDGET_SEC=60 ./ae.sh smoke-guider all
BUDGET_SEC=60 ./ae.sh smoke-recursive-rl-oac all
```

These commands check multi-head PPO trajectory collection, policy updates, and checkpoint generation; the Guider daemon, QASM/node IPC top-10 response, and soft-bias-assisted exploration; and recursive RL-OAC partitioning, leaf and boundary optimization, recomposition, and output-QASM validation. They are functional component checks, not independent measurements of each component's performance contribution.

### 4. Preview and check the reduced reproduction plan

```bash
./ae.sh reproduce plan
./ae.sh reproduce preflight
```

Use these commands to inspect the configured reduced matrix and check readiness before starting the full AE sequence.

### 5. Reproduce the reduced AE matrix

Estimated budget: **29.40 aggregate GPU-hours**, as described above. This sequence covers the selected 10 TDG circuits from Table IV, the Figures 6 and 8 ablations, and the representative long Nam comparison from Table V.

Start the resumable sequence in a detached terminal:

```bash
tmux new-session -d -s guiderq-reproduce \
  'cd /home/cc/guiderq/latest_ae && \
   mkdir -p results/reproduced && \
   LATEST_AE_PYTHON=/home/cc/quartz-ae/env/bin/python \
   AE_GPUS="0 1 2 3" ./ae.sh reproduce-all \
     --run-root results/reproduced 2>&1 | \
     tee -a results/reproduced/reproduce_all.log'
```

The complete console log is appended to `results/reproduced/reproduce_all.log`, preserving earlier console output when a run is resumed. After all tasks finish, the workflow automatically summarizes logs and generates CSV, JSON, table, and figure outputs.

To resume after an interruption, reuse the same run root. If the previous detached session has ended, rerun the command above. If the session still exists, attach to it with `tmux attach-session -t guiderq-reproduce`; once the previous reproduction process has stopped, run:

```bash
cd /home/cc/guiderq/latest_ae
LATEST_AE_PYTHON=/home/cc/quartz-ae/env/bin/python \
AE_GPUS="0 1 2 3" ./ae.sh reproduce-all --run-root results/reproduced
```

The workflow retains completed tasks and resumes the remaining work. Do not start concurrent reproduction processes against the same run root.

### 6. Inspect live progress

```bash
./ae.sh status --run-root results/reproduced
```

### 7. Locate and interpret generated results

Print the generated output paths:

```bash
./ae.sh results --run-root results/reproduced
```

| Location | Contents |
| --- | --- |
| `results/reproduced/summary/paper/` | Paper-format tables and convergence plots |
| `results/reproduced/summary/analysis/` | Supporting equal-quality, time-to-same-gate analyses |
| `results/reproduced/summary/table_v_long_representatives.csv` | Underlying long Nam comparison |
| `results/reproduced/tasks/` | Raw commands, logs, events, QASM files, and task status records |
| `results/reproduced/reproduce_all.log` | Reproduction console log |
| `reference_results/canonical_20260726/` | Bundled canonical reference run, left unmodified during execution |

The paper-output directory contains:

- `table_iv_tdg_reproduced.{tex,pdf,png}` for the selected 10 TDG circuits.
- `table_v_realamprandom_reproduced.{tex,pdf,png}` for the one-row representative long Nam comparison, including input, output, and reduction values.
- Convergence plots in PDF and PNG formats for the **Figures 6 and 8** ablations (`*convergence.pdf` and `*convergence.png`).

The bundled plotting scripts may retain legacy numbering in their output filenames. Use the exact paths printed by `./ae.sh results --run-root results/reproduced`; the paper references in this README use the camera-ready numbering, Figures 6 and 8.

The preparation results reported in the AE appendix provide reference expectations: approximately 32-33% gate-count reduction for GuiderQ versus 30-31% for Quarl on the selected 10 TDG circuits, and approximately 9-11% versus 1-3% on `realamprandom_indep_130`. These values describe the reduced AE setting and are not guarantees for every run or full-paper aggregate results. Compare both methods using their shared preprocessed input and the input/output counts in the generated results.

For the Figures 6 and 8 experiments, intrinsic stochasticity in RL exploration can introduce per-run variation even with fixed seeds. The time-to-same-gate measurements provide **complementary evidence of the overall acceleration behavior observed in the AE runs**. They **do not independently isolate the Guider contribution** from other architectural differences and should not be interpreted as a standalone causal attribution to the Guider.

### 8. Recover events or refresh summaries (optional)

Reconstruct missing improvement events from preserved raw logs:

```bash
./ae.sh extract-events --run-root results/reproduced
```

After event recovery, manual result edits, or an interruption before the automatic final summary, regenerate CSV, JSON, table, figure, and index outputs:

```bash
./ae.sh summarize --run-root results/reproduced
```

