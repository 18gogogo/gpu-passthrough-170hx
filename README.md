# GPU Passthrough & Rental — 170HX (GA100 64GB) on KVM

A field-tested runbook for passing **4× CMP 170HX (GA100, 64GB-unlocked)** GPUs from an
Ubuntu host through to a single KVM/libvirt VM, plus the pitfalls that actually bit us.
Written from a real rig: MSI X399 SLI PLUS, 128GB RAM, R5 340X host display, headless host.

> This is a **Hermes Agent skill** (SKILL.md + references). You can read it as a normal
> markdown runbook, or drop the whole folder into `~/.hermes/skills/` and let Hermes load it.

## What this covers
- **Strategy A: runtime nvidia ↔ vfio switching** — the *override-first* order that actually
  works when something (rustdesk, a lingering session, nvidia-persistenced) keeps auto-loading
  the nvidia module and makes the naive "rmmod first" order fail with `device or resource busy`.
- **VM creation** (Q35, SeaBIOS, 64GB BAR, virtual GPU kept, swtpm TPM).
- **Guest verification** and the cmpunlocker 8GB→64GB unlock rule (*the unlock follows the
  driver, not the card*).
- **Diagnosing a faulty card vs a faulty slot** — GSP booter error codes (0x31 = normal noise
  on the patched driver; 0x15 + `Max GSP-RM boot attempts exceeded` = real fault), the
  card-swap test, and VM IP drift.
- **Headless host** — the VM's GNOME desktop (gdm3) is your only GUI; the host has no monitor.
- **Long-running guest tasks** (driver install) and **rental** notes (Vast.ai etc.).

## Files
| File | What it is |
|---|---|
| `SKILL.md` | The main runbook (start here). |
| `references/cmp-170hx.md` | Card facts: what the 170HX/GA100 actually is. |
| `references/cmpunlocker.md` | The 8GB→64GB unlock mechanism + install/verify steps. |
| `references/gpu-rental-platforms.md` | GPU rental platform facts. |

## Key standing rules (read these first)
1. **Never passthrough the host display card** — it is the host's only display output.
2. **170HX cards have NO display output** — the VM must keep a virtual GPU (QXL/virtio-gpu);
   you reach the VM via RDP / RustDesk / Tailscale. Never follow a guide that says
   "remove the default video device".
3. **4 cards go to ONE VM at a time.**
4. **Clone/backup before touching GRUB** (IOMMU kernel args) — the only step that can brick boot.
5. **Data Center (Tesla) driver branch** on host AND guest — GeForce/Quadro branches don't know GA100.

## Adapting to your rig
- Replace `<host-ip>` / `<host-user>` / `<vm-ip>` with your own values.
- The slot↔card map in SKILL.md is a *worked example* from this rig — re-verify after any swap.
- The canonical per-rig guide referenced is `/home/ubuntu/170hx.md` on the host (not included here).

## License
MIT — do what you want, no warranty. GPU passthrough can brick a boot if you get GRUB wrong;
back up first.
