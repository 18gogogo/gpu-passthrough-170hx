# Windows-native 170HX unlock tool (blog kkk.rs ga100ctl path)

Use when the user wants the Windows-native unlock inside a VM (or bare Windows) instead of / alongside cmpunlocker. Source: blog.kkk.rs/archives/70 — a Windows-native unlock tutorial, an AI-generated PoC that writes PCI config space / MMIO directly (no driver patch, no firmware write). Author-declared high risk (possible bricking) — treat the binary as untrusted.

## Relationship to cmpunlocker (the rules that matter)
- The unlock follows the DRIVER, not the card: a stock Windows guest driver re-initializes the GPU and resets the memory geometry, so the VM sees 8GB even though the host shows 64GB.
- VFIO passthrough gives the tool direct PCI access, so its writes reach the real card — mechanically it CAN unlock inside a Windows VM.
- Host cmpunlocker v0.4 gsp-restore restores GSP boot state on every vfio-pci bind, so the card returns to the host healthy; how the blog tool's written values interact with v0.4 state is UNVERIFIED — verify the host (4x 64GB via nvidia-smi) after the card comes back.
- Neither tool writes firmware/OTP: worst case is a cold power cycle, not a bricked card. Still run the FIRST attempt on ONE card only.

## File inventory (the user's 170HX (2).zip tool package, 117 files)
- NEEDED: 170_boot_v3.rar (the unlock tool), TestCert.exe (time-server certificate installer), DDU (driver-uninstall fallback), 170_boot.inf/.sys/.cat (manual driver-install aids).
- NOT needed: ga100ctl.log (old run log).
- PRIVACY RED LINE: the password note (开锁密码.rtf) — never open, never transmit, never upload when publishing. The tool password is public per the blog (kkk.rs).

## Blog constraints (verify before executing)
- Driver: official NVIDIA **Data Center (Tesla/A100) branch** only — try 596 then 580. The GeForce branch is NOT usable: it ships `nv_dispi.inf`, and ga100ctl's post-unlock restore-to-A100 step looks for `nv_dispwi.inf` (Data Center inf, device token 20F1). Wrong branch → `NVIDIA_A100_CANDIDATE_REJECT ... original_inf_match=0`, `NVIDIA_A100_PREPARE_END result=failed error=1168`, batch abort. TCC mode (not WDDM): Task Manager shows nothing, only nvidia-smi.
- Windows 11 prep: Secure Boot OFF (Confirm-SecureBootUEFI), Memory Integrity OFF, Smart App Control OFF; bcdedit TESTSIGNING ON only if signature errors persist. Blog also says disable ASPM in board UEFI (some users saw idle card drops) — in a VM that maps to host-board UEFI or the Q35 chipset setting, usually a no-op.
- Multi-card support not guaranteed (mixed NV platforms especially); 8G/10G mixing supported since v3.
- Verify the v3 rar against the blog original before use: blog.kkk.rs/upload/article/170_boot_v3.rar, 10,809,189 bytes.
- Signature check before running: signer OKASAN ONLINE SECURITIES CO.,LTD., 2021-03-01 (a repurposed certificate — untrusted; run only inside the VM).

## Completion state + log reading (verified on this rig, 8GB SKU → 64GB)
- Status: DONE — Data Center 596 driver in the Win11 VM + TestCert + ga100ctl (password `kkk.rs`) → user confirmed 64GB in-guest.
- Reading a ga100ctl run: the unlock itself is the `KMD_ABI_OK` → `ROP_PROFILE ... 8g_to_64g` → `IOCTL_RUN_COMPLETE rc=0` sequence. Everything after that (`NVIDIA_A100_PREPARE/INSTALL`, `PNP_RESTORE`) is the tool putting the card back under the real driver — a failure there (error 1168, error 183) means the WRITE already landed even though the run reports `rc=1`. Check `nvidia-smi` inside the VM before re-running anything; the tool may just not have reached its verify step.
- Order that works: DDU (clean uninstall) → install Data Center branch → TestCert.exe → ga100ctl.exe → reboot if prompted → `nvidia-smi` shows 65536 MiB (8GB SKU) or 40960 MiB (10GB SKU) in TCC mode.

## Ready-to-execute plan
- Plan file: `/home/ubuntu/ok/gpu/win11vm-plan.md` on the agent workspace machine (Win11 VM + 1-card passthrough, 6 phases; verified platform readiness: every 170HX in its own IOMMU group, kvm_amd loaded, swtpm + OVMF secboot present, ~460G free disk).
- Split config keeps clore renting the remaining 3 cards during the attempt; choose the card by CURRENT lspci, never the setup notes' BDF table (it predates card moves).
