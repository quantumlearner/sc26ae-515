# GuiderQ SC'26 Artifact

Welcome to the GuiderQ SC'26 Artifact Evaluation repository for paper 515. This README documents the latest Chameleon image and the launch instructions for the GuiderQ artifact.

## Current image information

| Field | Value |
| --- | --- |
| Image name | `sc26ae-515-latest-ae-20260727-v2` |
| Image UUID | `70fd7289-191e-4853-8835-3b8df92b6523` |
| Chameleon site | `CHI@UC` |
| Created | `2026-07-27T14:42:06Z` |
| Updated | `2026-07-27T14:44:08Z` |
| Status | `active` |
| Visibility | `shared` |
| Protected | `True` |
| Disk format | QCOW2 |
| Compressed size | 13,570,004,992 bytes (12.64 GiB) |
| Virtual size | 31,881,953,280 bytes |
| Checksum | `9b22807fd04f8f8153bb9383f6a644da` |
| Base system | Ubuntu 22.04 CUDA, x86-64 |

This image was validated on a four-GPU NVIDIA A100 80 GB PCIe bare-metal instance. You may use an x86 Chameleon node with A100 PCIe or A100 NVLink GPUs.

## Launch

1. Log in to Chameleon and open the CHI@UC dashboard.

   a. You can go directly to the leases page: https://chi.uc.chameleoncloud.org/project/leases/

   b. Click the "Host Calendar" button on the leases page. The calendar allows selecting a node type and viewing available times.

   c. Choose a GPU node type. We recommend `gpu_a100_pcie` because it is generally more available than the NVLink option.

   Example screenshots:

   ![Host calendar screenshot 1](https://github.com/user-attachments/assets/c5470d33-f7ba-4821-a98b-be277a5d9c8b)

   ![Host calendar screenshot 2](https://github.com/user-attachments/assets/39f71682-14e3-419e-b5af-da13e545012f)

   After selecting a time window (e.g., 30 days), choose an available slot and click the node name to create a lease.

   Please see the guide: https://chameleoncloud.readthedocs.io/en/latest/getting-started/index.html#step-3-reserve-a-node-directly-from-the-calendar

2. Locate the image by its exact name or UUID in the Chameleon image list.

   See the Chameleon documentation for guidance: https://chameleoncloud.readthedocs.io/en/latest/getting-started/index.html#my-first-instance-launching-an-instance
   <img width="1230" height="469" alt="image" src="https://github.com/user-attachments/assets/c6f2e0ee-2caf-493c-a10a-68da44ee851a" />



4. Launch an x86 bare-metal GPU instance with the required key pair and networking configuration.

5. Connect to the instance as user `cc`, then run the following commands:
## Artifact Evaluation commands

### 1. Set up and validate the artifact — Recommended

Estimated time: under one minute; 0 GPU-hours.

```bash
cd /home/cc/guiderq/latest_ae
export LATEST_AE_PYTHON=/home/cc/quartz-ae/env/bin/python
export AE_GPUS="0 1 2 3"

./ae.sh check
```

This validates Python, CUDA, manifests, checkpoints, benchmark inputs, bundled Quartz paths, and the locked ECC/actor action dimensions.

- The frozen `quartz-ppo` environment is located at `/home/cc/quartz-ae/env`.
- The standalone artifact, locked checkpoints, circuits, scripts, and reference results are located at `/home/cc/guiderq/latest_ae`.

### 2. Run all Functional/Reusable smoke tests — Recommended

Estimated time: up to 10 minutes of wall time on four A100 GPUs; up to 0.67 GPU-hours.

```bash
./ae.sh smoke-all
```

This is the default end-to-end smoke test for contributions C1, C2, and C3. It runs four concurrent GPU lanes and writes verified logs, final checkpoints, and QASM outputs to:

```text
results/history/smoke-runs/full_parallel_<timestamp>/
```

### 3. Test individual contributions — Optional

The following commands use one GPU by default and process the TDG, IBM, and Nam gate sets sequentially.

#### C1: Compact global representation

Estimated time: approximately three minutes; approximately 0.05 GPU-hours.

```bash
BUDGET_SEC=60 ./ae.sh smoke-multihead all
```

This isolates multi-head PPO trajectory collection, policy updates, and final-checkpoint generation.

#### C2: Guided exploration

Estimated time: approximately three minutes; approximately 0.05 GPU-hours.

```bash
BUDGET_SEC=60 ./ae.sh smoke-guider all
```

This isolates the Guider daemon, QASM/node IPC top-10 response, and soft-bias-assisted multi-head PPO.

#### C3: Scalable segment optimization

Estimated budget: approximately nine single-GPU minutes in total; approximately 0.15 GPU-hours.

```bash
BUDGET_SEC=60 ./ae.sh smoke-recursive-rl-oac all
```

This isolates recursive partitioning, leaf-circuit PPO, meld-boundary PPO, recomposition, and output-QASM validation.

### 4. Reproduce Table IV, Figures 5–6, and the long Nam representative — Recommended

Estimated cost: 30.33 aggregate GPU-hours. The ideal four-GPU wall time is 7.58 hours; reserve up to eight hours for launch overhead, validation, and queue imbalance.

Start the resumable fixed-seed 28-task sequence in a detached terminal:

```bash
tmux new-session -d -s guiderq-reproduce \
 'cd /home/cc/guiderq/latest_ae && \
  mkdir -p results/reproduced && \
  LATEST_AE_PYTHON=/home/cc/quartz-ae/env/bin/python \
  AE_GPUS="0 1 2 3" ./ae.sh reproduce-all \
   --run-root results/reproduced 2>&1 | \
   tee results/reproduced/reproduce_all.log'
```

The matrix covers the ten-circuit TDG subset of Table IV, the Figure 5–6 ablations, and the paired three-hour RL-OAC comparison on `realamprandom_indep_130`.

Completed tasks are resumable: run the same command again after an interruption. After all tasks finish, the command automatically extracts results, summarizes logs, and generates the paper-format tables and figures.

### 5. Inspect live progress

```bash
./ae.sh status --run-root results/reproduced
```

### 6. Locate generated results

```bash
./ae.sh results --run-root results/reproduced
```

Primary paper-format outputs are generated under:

```text
results/reproduced/summary/paper/
```

This directory includes:

- `table_iv_tdg_reproduced.{tex,pdf,png}`
- `table_v_realamprandom_reproduced.{tex,pdf,png}`
- `figure5_convergence.{pdf,png}`
- `figure6_convergence.{pdf,png}`

Supporting equal-quality timing analyses are under:

```text
results/reproduced/summary/analysis/
```

Raw commands, logs, events, QASM files, and task status records are under:

```text
results/reproduced/tasks/
```

### 7. Recover events or refresh summaries — Optional

Reconstruct improvement events from preserved raw logs:

```bash
./ae.sh extract-events --run-root results/reproduced
```

Regenerate all CSV, JSON, table, figure, and index outputs after event recovery, manual result changes, or an interruption before the automatic final summary:

```bash
./ae.sh summarize --run-root results/reproduced
```

These quick tests verify that the artifact is functional and usable.

- The frozen `quartz-ppo` environment is located at `/home/cc/quartz-ae/env`.
- The standalone artifact, locked checkpoints, circuits, scripts, and reference results are located at `/home/cc/guiderq/latest_ae`.

To preview or run the reduced artifact reproduction experiment:

```bash
./ae.sh reproduce plan
./ae.sh reproduce preflight
./ae.sh reproduce-all --run-root results/reproduced
```

Progress and generated paper-format outputs can be inspected with:

```bash
./ae.sh status --run-root results/reproduced
./ae.sh summarize --run-root results/reproduced
```
