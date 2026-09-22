# GPU Passthrough & Rental — CMP 170HX (170HX) rig

A practical, battle-tested operations manual for running **4× NVIDIA CMP 170HX (GA100, 64GB-unlocked)** on a single consumer workstation:

- **Docker GPU rental** (Clore.ai) with the 3-4 cards left on the host
- **PCIe passthrough** of 1 or 4 cards to VMs (virt-manager / libvirt + QEMU)
- **Windows 11 unlock path** (blog kkk.rs `ga100ctl`) — 64GB verified
- Host desktop survival, RustDesk, vLLM coexistence, and a full fault/restore playbook

Written for the author's rig (MSI X399, R5 340X display, Ubuntu 26.04 desktop) but organized so the transferable lessons (fd-holder hunting, override-first switching, locale traps) are easy to lift.

## Contents

| File | What it covers |
|---|---|
| `SKILL.md` | Main manual: standing rules, switch flows (full 4-card + partial single-card), VM creation, guest verification, rental ops, restore |
| `references/cmp-170hx.md` | Card facts: SKUs, unlock mechanism, driver branches |
| `references/cmpunlocker.md` | The Linux-side unlock (cmpunlocker) + GSP boot state |
| `references/faulty-card-diagnosis.md` | Fault history, card-vs-slot isolation, current slot map |
| `references/clore-rental.md` | Clore.ai agent setup, container whitelist, pricing workflow |
| `references/gpu-rental-platforms.md` | Platform comparison |
| `references/guest-remote-desktop.md` | RustDesk inside the guest |
| `references/desktop-and-boot.md` | Headless host desktop, boot triage, worked restore examples |
| `references/windows-unlock-tool.md` | The Windows-native ga100ctl unlock (blog kkk.rs) |

## Key gotchas (the ones that cost real hours)

1. **Every process holding `/dev/nvidia*` blocks unbind** — vLLM with `NVIDIA_VISIBLE_DEVICES=all`, the Clore agent, `ptyxis`, even RustDesk. Find them with `sudo lsof /dev/nvidia*` BEFORE any switch.
2. **Override-first** is the only reliable unbind/bind order.
3. **D-state wedges are unkillable** — reboot is the only fix; wrap every sysfs write in `timeout`.
4. **Localized libvirt** — state names come back in Chinese (執行中/關機); never grep for English.
5. **BDF maps rot** — after any physical card move, re-derive from `lspci -nnk` and re-check the scripts.
6. **Secure Boot off + Above 4G Decoding** (or QEMU `pci-hole64-size=1TB`) for 64GB-BAR guests.

## License

MIT — take what helps, fix what doesn't, and file a PR with the card you tested it on.
