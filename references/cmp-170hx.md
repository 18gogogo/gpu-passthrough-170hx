# CMP 170HX card facts (verified)

- Chip: GA100 (Ampere, A100 silicon), 2021. NOT GP107, NOT a GeForce card.
- `lspci`: `GA100 [CMP 170HX] [10de:20c2]` (8/16/64GB SKUs) or `[10de:2082]` (10GB), class `3D controller [0302]`, rev a1.
- ONE PCI function per card — no audio device, no second function. GPU device lists hold 4 entries, not 8.
- No display output (mining card). VMs need a virtual display + remote desktop (RDP/RustDesk).
- HBM2e memory, factory-limited 8/10GB; community VBIOS unlocks 16/64GB (nvflash — back up stock ROM first; analysis project: Consensus-Protocol/cmp170hx on GitHub).
- Factory PCIe limited to 1.0 x4 (some unlocked mods); multi-GPU bandwidth bottleneck, single-GPU workloads fine.
- Driver: Data Center (Tesla) branch required on host AND inside VMs; GeForce branch won't recognize GA100.
- No Error 43 (GeForce-only phenomenon); hyperv tuning still recommended for Windows VMs.
- Modded cards may report non-stock device IDs — always trust live `lspci -nn` output over any doc.

## Per-card identity & health profile (verified 2026-10-02, live rig)

| Slot (BDF) | nvidia-smi | Serial | Batch | Time Run (odometer) | Health under identical 100W mining load |
|---|---|---|---|---|---|
| 0000:0a:00.0 | GPU 0 | 1322421x… | 1322421x | 124.7日 | 82.4 TH/s, 53°C, 823.6 GH/W |
| 0000:0b:00.0 | GPU 1 | 1322421x… | 1322421x | 155.8日 (highest) | 79.2 TH/s, 56°C, 792.5 GH/W |
| 0000:42:00.0 | GPU 2 | 1322421x… | 1322421x | 138.3日 | 74.8 TH/s, 57°C, 748.3 GH/W (weakest — silicon lottery) |
| 0000:43:00.0 | GPU 3 | 1322621x… | **1322621x (different batch)** | 96.0日 (lowest) | **85.0 TH/s, 53°C, 858.6 GH/W (strongest)** |

- The user refers to cards by SLOT bus number ("0a卡/43卡"), NOT by nvidia-smi index — always map both ways before touching anything.
- Serial prefix = production batch: 1322421x (0a/0b/42) vs 1322621x (43). The different-batch card (43) is coincidentally the strongest miner and has the least prior use.
- Time Run (Inforom) is cumulative since manufacture — it survives reboots, unlike `nvidia-smi -pl 100` power caps.
- Hashrate spread (74.8–85.0 TH/s at identical 100W) is silicon lottery, NOT wear: no correlation between hashrate and odometer.
- All 4 run 64GB unlocked (cmpunlocker). Total ~321.5 TH/s at ~398W when PRL mining.
