# GPU rental platforms — host-side facts (verified)

Model: register machine as host → set price → platform auto-starts/stops containers per rental. No long-lived rental daemons. The old guide's `vastai/vastai-host` and `runpod/runpod-host` images do NOT exist.

## Vast.ai
- Hard requirements: public IPv4 + port forwarding (5 ports per GPU, 100 recommended; Tailscale is NOT a public IP), ≥200GB dedicated Docker SSD partition, ≥20GB free on root, wired ≥500Mbps up/down, ≥4GB RAM per GPU (8 recommended), >2.85 GiB/s PCIe per GPU.
- Official images (pulled by platform per rental): `vastai/base-image`, `vastai/pytorch`.
- Host setup: https://cloud.vast.ai/host/setup (vastai CLI → host account → register machine → price/list).

## RunPod
- Community Cloud (self-hosted GPU machines) requires application/approval — not self-serve.
- Official base image: `runpod/base`.

## Clore.ai
- Marketplace API (no auth): `curl -s https://clore.ai/webapi/marketplace/servers` → `all_servers[]`; `specs.gpu` ("N× model"), `price.on_demand.USD-Blockchain` = whole-server USD/DAY, `price.spot.USD-Blockchain`, `rented`, `partial_gpu_rental.free_indices`, `reliability`, `specs.pl` (per-card power-limit array). Per-GPU-hr = daily price / n / 24.
- Fees: spot 1.8-2.5%, on-demand 5-10% (off your listed price).
- Modded mining cards (170HX): VERIFIED listable and rentable — market median ~$0.20-0.22/GPU/hr (vs 4090 ~$0.42, 5090 ~$0.60); demand is low (only 3 such listings, 0% rented at snapshot time). Pricing is set in the web dashboard (Server price → on-demand + min spot), not by host command.
- Partial rental (some cards user-held, rest listed) works — ~20% of market servers do this.
- Hardware notes from agent source (lib/get_specs.py): system RAM is a listing field but does NOT move GPU prices (market data: 4090 median identical at 64/128GB RAM; 256GB slightly lower) — don't recommend RAM upgrades for rental revenue. Disk model + measured read/write speed are displayed to renters and affect listing appeal indirectly (capacity/speed, not interface type); the agent re-detects them at every heartbeat, so disk upgrades need no re-registration.

## Rules
- One card = one platform at a time. Unlist from A before listing on B.
- Rental only in Docker (nvidia) mode; switch to vfio before VM rentals.
- Modded mining cards (170HX): listed + rentable on Clore at ~$0.20/GPU/hr (see Clore section); expect low demand; PCIe 1.0 x4 may fail multi-GPU workloads.
