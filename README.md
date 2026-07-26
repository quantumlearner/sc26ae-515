# GuiderQ SC'26 Artifact

This repository records the current Chameleon image and launch instructions
for the GuiderQ artifact.

## Current image

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

1. Open the CHI@UC Chameleon dashboard.
2. Locate the image by its exact name or UUID.
3. Launch an x86 bare-metal GPU instance with the required key pair and
   networking configuration.
4. Connect as user `cc`, then run:

```bash
cd /home/cc/guiderq/latest_ae
export LATEST_AE_PYTHON=/home/cc/quartz-ae/env/bin/python
export AE_GPUS="0 1 2 3"

./ae.sh check
./ae.sh smoke-all
```

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
```

The image upload completed successfully, its Glance status is `active`, and
the uploaded checksum matches the locally generated QCOW2 file.
