# cmpunlocker — 170HX 8GB→64GB unlock

## What it is
The 170HX is physically an 8GB HBM2e card; the GA100 die supports 64GB and NVIDIA locks it via OTP fuses + firmware. `cmpunlocker` (github.com/amoghmunikote/cmpunlocker) does NOT touch firmware — it patches the nvidia-open kernel module so that at driver init it opens the PLM registers and writes the memory-geometry registers (CFG1/LMR) + SM throttle registers (SS0/SS1) BEFORE GSP-RM boots. Patched modules live in `/lib/modules/$(uname -r)/updates/cmpunlocker/` and run on every driver init, so the unlock persists across reboots.

## The rule that matters for passthrough
**The unlock follows the driver, not the card.** Whichever driver initializes the GPU must be the patched one:
- Host patched → host `nvidia-smi` shows 64GB.
- VM guest with stock driver → guest sees **8GB** (the guest driver re-initializes the GPU and resets the registers).
- VM guest with patched driver + cold reboot → **64GB**.

## Install requirements (host or guest)
- Linux x86-64, root, a supported stock nvidia-open driver already installed (supported versions are listed in cmpunlocker's `driver/VERSION` — 610.57.04 on this rig), kernel headers for the running kernel, Secure Boot disabled (patched modules are unsigned), network on first install.
- Cold reboot after install (full power cycle / `virsh destroy && virsh start` for a VM — a warm `reboot` is not enough).
- Installer adds `iommu=pt` by default (negligible overhead, required for VM passthrough).
- The patched module is built against ONE kernel: after any kernel or driver change, rebuild/reinstall, or the card silently comes back at 8GB.

## Verify (any machine)
```bash
nvidia-smi --query-gpu=index,memory.total --format=csv   # 65536 MiB = unlocked
modinfo -n nvidia    # path must contain updates/cmpunlocker
ls /lib/modules/$(uname -r)/updates/cmpunlocker          # card_profile, unlock_geometry, nvidia.ko
sudo dmesg | grep -iE "SEC2_DEBUG|WPR meta|Booter"       # fbSize=0x...1000000000 = 64GB
```

## VM guest specifics
- Install the patched driver INSIDE the guest, then cold reboot the VM.
- After any in-guest kernel upgrade, reinstall the patched driver (stock kernel update drops the unlock).
- Fresh-guest prep BEFORE the stock driver: blacklist + unload nouveau (Ubuntu auto-loads nouveau on the passed-through GA100 cards and the nvidia module cannot load while nouveau owns the card) and stop gdm3 (desktop guest; the 170HX has no display output so GNOME runs on the virtual GPU — stopping it is safe and keeps the .run installer from stalling on X).
- The 610.57.04 .run installer rejects `--no-persistenced` (unrecognized option → exit 1 → no firmware installed → cmpunlocker's version check fails). Verify flags against the actual installer, not by grepping the .run binary — the binary contains flag strings in help/error text, so grep false-positives.
- If the stock driver is missing, cmpunlocker's version detection falls back to the generic `/lib/firmware/nvidia/` directory and reports a bogus version (e.g. "tu117"). "Installed driver is <garbage>" means the stock-driver step failed, not a version mismatch.
- `nvidia-smi -L` can hang on a faulty card and take the guest's SSH down with it — use `timeout 30 nvidia-smi --query-gpu=... --format=csv` instead.
- Multi-hop SSH from a client box: `ssh -J <host-user>@<host-ip> <user>@<vm-ip>`. Host→VM key auth is usually established during a rebuild (verify; re-establish after any VM rebuild) — the guest still needs a password for sudo. Base64-encode scripts before sending (multi-layer quoting mangles `(` `|` `>`).

## v0.4: VFIO passthrough support (fixes the guest GSP 0x15 card-drop)
- v0.3 lacked passthrough support: binding a card to vfio-pci lost its GSP boot-time registers, so the guest's GSP could not boot (Booter error 0x15) and exactly one card dropped per guest boot (which card varied — NOT a dead card, NOT a slot fault).
- v0.4 adds three components (verify all present after install):
  - `cmp_no_bus_reset.ko` in `/lib/modules/$(uname -r)/updates/cmpunlocker/` — blocks bus reset so the unlock survives the handoff to the VM.
  - udev rule `/etc/udev/rules.d/99-cmpunlocker-passthrough.rules` + `/usr/local/lib/cmpunlocker/gsp-restore` — on every vfio-pci bind restores the GSP boot-time registers so the guest can boot GSP. Verify in journal: `sudo journalctl | grep cmpunlocker` should show a 'GSP boot state ... clean' line per bind.
  - `cmpunlocker-passthrough.service` (enabled) — arms all CMP cards at boot (persistence off, reset_method cleared, cmp_no_bus_reset loaded). Arming is NON-destructive: it never unbinds cards from nvidia, so a split config (some cards on the host) is safe.
- `--no-passthrough` install flag skips the VM prep. v0.4 also drops the cold-reboot requirement ('No more cold reboots').
- Manual tools: `sudo ./tools/passthrough.sh status|prepare|restore <bdf...>` — `restore` is needed after a VM was KILLED (not shut down): the ACR version stamp is write-protected once set and blocks the next VM until cleared.
- Verified on this rig (2026-09-20): all 4 cards → one VM, guest shows 4x 64GB, only benign 0x31 Booter noise.

## Known-good state on this rig (verify, don't assume)
- Host: cmpunlocker v0.4 installed (passthrough components present), `card_profile`=8gb, driver 610.57.04, 4x 64GB.
- User's notes: `/home/ubuntu/GPU-VM-VAST-SETUP-NOTES.md` on the host (VM names/IPs, credentials conventions, past pitfalls).
