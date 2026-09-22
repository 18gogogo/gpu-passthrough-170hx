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
- Driver: official NVIDIA only, try 596 then 580; 170HX is not in the default INF list — install the driver manually after the unlock.
- TCC mode (not WDDM): Task Manager shows nothing, only nvidia-smi.
- Windows 11 prep: Secure Boot OFF (Confirm-SecureBootUEFI), Memory Integrity OFF, Smart App Control OFF; bcdedit TESTSIGNING ON only if signature errors persist.
- Multi-card support not guaranteed (mixed NV platforms especially); 8G/10G mixing supported since v3.
- Verify the v3 rar against the blog original before use: blog.kkk.rs/upload/article/170_boot_v3.rar, 10,809,189 bytes.
- Signature check before running: signer OKASAN ONLINE SECURITIES CO.,LTD., 2021-03-01 (a repurposed certificate — untrusted; run only inside the VM).

## Ready-to-execute plan
- Plan file: `/home/ubuntu/ok/gpu/win11vm-plan.md` on the agent workspace machine (Win11 VM + 1-card passthrough, 6 phases; verified platform readiness: every 170HX in its own IOMMU group, kvm_amd loaded, swtpm + OVMF secboot present, ~460G free disk).
- Split config keeps clore renting the remaining 3 cards during the attempt; choose the card by CURRENT lspci, never the setup notes' BDF table (it predates card moves).
