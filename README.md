# GuiderQ SC'26 Artifact

Welcome to the GuiderQ SC'26 Artifact Evaluation repository for paper 515. This README records the latest Chameleon image and the launch instructions for the GuiderQ artifact.

## Current image information

| Field | Value |
| --- | --- |
| Image name | `sc26ae-515-latest-ae-20260727` |
| Image UUID | `13a4c1a4-6d7f-48f0-9e25-e316acbb9db0` |
| Chameleon site | `CHI@UC` |
| Created | `2026-07-26T16:56:17Z` |
| Status | `active` |
| Visibility | `shared` |
| Disk format | QCOW2 with zstd compression |
| Compressed size | 13,569,766,912 bytes (12.64 GiB) |
| Virtual size | 31,884,050,432 bytes (29.69 GiB) |
| Checksum (MD5) | `51297f1d0b79200f3a2fe8c191f24659` |
| Base system | Ubuntu 22.04 CUDA, x86-64 |

This image was validated on a four-GPU NVIDIA A100 80 GB PCIe bare-metal instance. An x86 Chameleon node with A100 PCIe or A100 NVLink GPUs may be used.

## Launch

1. Log in to the Chameleon platform and open the CHI@UC Chameleon dashboard.

   a. You can go directly to the leases page: https://chi.uc.chameleoncloud.org/project/leases/

   b. Click the "Host Calendar" (Host calendar) button on the leases page. The calendar allows selecting a node type and viewing available times.

   c. Choose a GPU node type. We recommend `gpu_a100_pcie` because it is generally more available than the NVLink option.

   Example screenshots:

   ![Host calendar screenshot 1](https://github.com/user-attachments/assets/c5470d33-f7ba-4821-a98b-be277a5d9c8b)

   ![Host calendar screenshot 2](https://github.com/user-attachments/assets/39f71682-14e3-419e-b5af-da13e545012f)

   After selecting a time window (e.g., 30 days), choose an available slot and click the node name to create a lease.

   ![Lease selection screenshot 1](https://github.com/user-attachments/assets/f281cd58-24b3-4be4-8afb-9f9809428d51)

   ![Lease selection screenshot 2](https://github.com/user-attachments/assets/7934e43a-c2b6-487a-b464-4cc9442dbac1)

   ![Lease creation screenshot](https://github.com/user-attachments/assets/02cc7cd6-6f46-467d-9642-07a738196063)

2. Locate the image by its exact name or UUID in the Chameleon image list.

   See the Chameleon docs for guidance: https://chameleoncloud.readthedocs.io/en/latest/getting-started/index.html#my-first-instance-launching-an-instance

3. Launch an x86 bare-metal GPU instance with the required key pair and networking configuration.

4. Connect to the instance as user `cc`, then run the following commands:

```bash
cd /home/cc/guiderq/latest_ae
export LATEST_AE_PYTHON=/home/cc/quartz-ae/env/bin/python
export AE_GPUS="0 1 2 3"

./ae.sh check
./ae.sh smoke-all
```

These quick tests verify that the artifact is functional and usable.

- The frozen `quartz-ppo` environment is located at `/home/cc/quartz-ae/env`.
- The standalone artifact, locked checkpoints, circuits, scripts, and reference results are located at `/home/cc/guiderq/latest_ae`.

To preview or run the reduced Artifact-Reproduced experiment:

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
