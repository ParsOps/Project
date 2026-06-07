# Our Customer Linux Hardening Report

**Date:** June 2, 2026
**Host:** Our Customer Linux

## Summary

All observed OS hardening tasks in the provided logs completed without task failures. The log entries show multiple system, account, filesystem and runtime hardening changes applied. The report below summarizes the specific options and areas hardened.

## Hardened Areas and Changes

- **Audit & Logging**: Installed and configured `auditd`; disabled `systemd-journald.audit`; hardened audit log directory permissions.

- **Cron & Scheduled Jobs**: Permissions enforced for `/etc/crontab` and cron directories (`/etc/cron.daily`, `/etc/cron.weekly`, `/etc/cron.monthly`, `/etc/cron.hourly`, `/etc/cron.d`) to reduce privilege exposure.

- **Core Dumps & Limits**: Created `coredump.conf.d` and custom config to disable coredumps; added `10.hardcore.conf` in `/etc/security/limits.d` with strict permissions (0400) and root ownership.

- **Account & Password Policy**:
  - Shadow and group shadow files verified at secure permissions (owner `root`, group `shadow`, mode `0640`).
  - `/etc/passwd` and `/etc/group` verified at `0644`.
  - Installed and configured `passwdqc` for stronger password checks.
  - Configured `faillock` (account lockout) and enabled corresponding PAM modules; removed or replaced legacy `tally2` artifacts.
  - Removed PAM password-caching module (`pam_ccreds`) where applicable.

- **Authentication / SSH**: Allowed login with SSH keys when user password is expired (SSH key fallback preserved in PAM flow as configured by the role).

- **Filesystem & Mount Options**:
  - Set `hidepid=2` on `/proc` to reduce process visibility between users.
  - Hardened mount options and directory permissions for key mountpoints (examples seen: `/dev` with `nosuid,noexec`, `/dev/shm` with `nosuid,nodev,noexec`, `/run` with `nosuid,nodev`).
  - Hardened permissions for `/boot` (mode `0700` where applicable) and `/var/log/audit` (mode `0700`).

- **SUID/SGID Hardening**: Removed suid/sgid bits from a set of legacy or risky binaries; notable change: `/usr/lib/openssh/ssh-keysign` had its suid/sgid cleared.

- **Kernel & Module Controls**:
  - Disabled unused filesystem modules via `modprobe` (package `kmod` used).
  - Applied sysctl customizations observed in variables (examples in logs: `kernel.unprivileged_userns_clone=0`, `kernel.unprivileged_bpf_disabled=1`).
  - Playbook variable shows `net.ipv4.ip_forward: 1` configured in play vars.

- **System Profile & Session**:
  - Added profile script (`pinerolo_profile.sh`) to `/etc/profile.d` and configured an `autologout` setting.
  - Created or ensured `securetty` exists.

- **Miscellaneous**:
  - Added tally/lock directories and ensured ownership and permissions.
  - Installed or ensured presence of supporting packages (e.g., `auditd`, `passwdqc`, `kmod`).

## Observations

- The logs show numerous `changed` tasks and a larger number of `ok` verifications; no failed tasks are present in the provided logs.

## Recommendations

- Review `/etc/ssh/sshd_config` and current PAM stack to validate SSH hardening settings match policy expectations.
- Test `faillock` behavior by simulating failed logins in a controlled environment to confirm lockout thresholds.
- Re-run the playbook periodically (or in CI) to ensure idempotency and detect drift.

---

_Report generated from provided logs on June 2, 2026._  
_Produced by [ParsOps](https://parsops.com) team._
