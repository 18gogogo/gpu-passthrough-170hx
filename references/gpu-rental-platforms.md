# GPU rental platforms — host-side facts (verified)

Model: register machine as host → set price → platform auto-starts/stops containers per rental. No long-lived rental daemons. The old guide's `vastai/vastai-host` and `runpod/runpod-host` images do NOT exist.

## Vast.ai
- Hard requirements: public IPv4 + port forwarding (5 ports per GPU, 100 recommended; Tailscale is NOT a public IP), ≥200GB dedicated Docker SSD partition, ≥20GB free on root, wired ≥500Mbps up/down, ≥4GB RAM per GPU (8 recommended), >2.85 GiB/s PCIe per GPU.
- Official images (pulled by platform per rental): `vastai/base-image`, `vastai/pytorch`.
- Host setup: https://cloud.vast.ai/host/setup (vastai CLI → host account → register machine → price/list).

## RunPod
- Community Cloud (self-hosted GPU machines) requires application/approval — not self-serve.
- Official base image: `runpod/base`.

## Rules
- One card = one platform at a time. Unlist from A before listing on B.
- Rental only in Docker (nvidia) mode; switch to vfio before VM rentals.
- Modded mining cards (170HX): acceptance/pricing untested — run platform self-tests, expect low demand; PCIe 1.0 x4 may fail multi-GPU workloads.
