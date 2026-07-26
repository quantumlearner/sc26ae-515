# GuiderQ SC'26 Artifact

Welcome to sc26 paper 515 Aritifact Evaluation! 
This repository records the latest Chameleon image and the latest launch instructions
for the GuiderQ artifact.

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

The image was validated on a four-GPU NVIDIA A100 80 GB PCIe bare-metal
instance. An x86 Chameleon node with A100 PCIe or A100 NVLink GPUs may be used.

## Launch

1. Login to Chamelon platform and Open the CHI@UC Chameleon dashboard.
   a.You could directly use this link to visit the lease page: https://chi.uc.chameleoncloud.org/project/leases/
   and
   b.then you could click <img width="1207" height="306" alt="image" src="https://github.com/user-attachments/assets/c5470d33-f7ba-4821-a98b-be277a5d9c8b" /> the host calendar button.
   c.on the host calendar button, you could click the node type button and choose gpu_a100_pcie. I recommend gpu_a100_pcie here because it is more availiable than the nvlink one. <img width="1071" height="392" alt="image" src="https://github.com/user-attachments/assets/b9819d16-fa7c-4ae5-a683-3a7a95d5d999" /> And you could change the time by 1day/7day/30days to see when there will be avaliable time.
   <img width="1190" height="588" alt="image" src="https://github.com/user-attachments/assets/39f71682-14e3-419e-b5af-da13e545012f" />
As the image shows, after choose 30 days, you could see there are some avaliable time in each node. And then you could choose one and click the node name in this page directly to make a lease reversation.Here is a example:
<img width="959" height="703" alt="image" src="https://github.com/user-attachments/assets/f281cd58-24b3-4be4-8afb-9f9809428d51" />
<img width="733" height="545" alt="image" src="https://github.com/user-attachments/assets/7934e43a-c2b6-487a-b464-4cc9442dbac1" />
<img width="731" height="571" alt="image" src="https://github.com/user-attachments/assets/02cc7cd6-6f46-467d-9642-07a738196063" />
and click create a lease successfully

2. Locate the image by its exact name or UUID.
   Please visit this page for guide.
   https://chameleoncloud.readthedocs.io/en/latest/getting-started/index.html#my-first-instance-launching-an-instance
4. Launch an x86 bare-metal GPU instance with the required key pair and
   networking configuration.
5. Connect as user `cc`, then run:

```bash
cd /home/cc/guiderq/latest_ae
export LATEST_AE_PYTHON=/home/cc/quartz-ae/env/bin/python
export AE_GPUS="0 1 2 3"

./ae.sh check
./ae.sh smoke-all
```
This is a quick test to verify that the artifact is functional and usable
The frozen `quartz-ppo` environment is located at
`/home/cc/quartz-ae/env`. The standalone artifact, locked checkpoints,
circuits, scripts, and reference results are located at
`/home/cc/guiderq/latest_ae`.

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
``
