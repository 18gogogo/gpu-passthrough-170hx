# Guest remote desktop (RustDesk) setup for the headless-rig VM

The host is headless (no monitor on the R5 340X) — the VM's GNOME desktop is the only GUI, and RustDesk inside the guest is how the user reaches it. This recipe was verified on the Ubuntu 24.04 desktop guest.

## Install (Ubuntu 24.04 guest)
- `https://rustdesk.com/install.sh` is DEAD (returns a 404 HTML page — the site moved). Use the official GitHub release .deb instead:
  - Latest tag: `curl -s https://api.github.com/repos/rustdesk/rustdesk/releases/latest | grep tag_name`
  - `curl -sL -o /tmp/rustdesk.deb https://github.com/rustdesk/rustdesk/releases/download/<ver>/rustdesk-<ver>-x86_64.deb`
  - `sudo dpkg -i /tmp/rustdesk.deb && sudo apt-get install -f -y` (the .deb needs libva*/libxdo4; `apt -f` pulls them; `dpkg -i` alone leaves the package half-configured)
- The service runs as root; config lives in `/root/.config/rustdesk/RustDesk.toml` (capital R — `cat /root/.config/rustdesk/rustdesk2.toml` finds nothing useful).
- Set a password AT INSTALL TIME — a fresh install has an empty password and connections are rejected: `sudo rustdesk --password <pw>`.
- `systemctl enable --now rustdesk`; verify `systemctl is-active rustdesk` + UDP 21119 listening (`sudo ss -ltnup | grep 2111`).

## GDM autologin (required)
Without autologin the guest sits at the login screen (greeter) and RustDesk has no user session to share:
- `/etc/gdm3/custom.conf`:
  ```
  [daemon]
  AutomaticLoginEnable=true
  AutomaticLogin=<user>
  ```
- `sudo systemctl restart gdm3`, then verify the guest is still up (ping + `virsh domstats`) — a gdm3 restart in a passthrough VM has been observed to crash the guest; recover with `virsh destroy && virsh start`.
- Verify a REAL user session (not the greeter): `loginctl list-sessions` shows a seat0 session with `Type=x11` (or wayland) and `ps -C gnome-shell` is owned by the user, not gdm.

## The RustDesk ID
- The ID is derived from the guest's primary MAC (official hbb_common `get_auto_id`): build a u32 from the LAST 3 BYTES of the MAC, mask with `0x1FFFFFFF`, print in decimal.
  - Worked example: MAC `52:54:00:24:04:50` → ID `2360400`.
- The VM's MAC is fixed in the libvirt definition → the ID is STABLE across reboots and survives driver/GPU changes; give the user the ID once and it keeps working.
- Do NOT rely on `rustdesk --id` — it launches the flutter GUI and spits debug logs (`flutter: launch args...`) with no clean ID on stdout.
- The config's `enc_id` is an encrypted blob, not the ID; `key_pair` is the ed25519 pair, not the ID either. The MAC formula is the reliable source.

## Troubleshooting
- "Can't connect" while SSH works, in order: (1) is the client dialing the HOST's RustDesk ID? The host's RustDesk shares nothing (headless). (2) `systemctl is-active rustdesk` inside the guest — not-found means it was never installed (typical after a VM rebuild). (3) Is a password set? (4) Is there a live user session (autologin configured)?
- Guest dead (ping 100% loss, `virsh domstats` state.reason=1 CRASHED, qemu process still alive): `virsh destroy <vm> && virsh start <vm>` — the GPUs re-attach automatically from the XML hostdevs; no re-binding needed.
