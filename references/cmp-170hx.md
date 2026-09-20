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
