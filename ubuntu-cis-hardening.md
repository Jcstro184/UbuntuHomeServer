# Automated Ubuntu Server CIS Hardening (Ansible + Lynis)

An Infrastructure-as-Code (IaC) security project using **Ansible** from a **Kali Linux** control node to audit, harden, and validate an **Ubuntu 26.04** Linux server against CIS Benchmarks and **Lynis** security recommendations.

## Architecture & Workflow
* **Control Node:** Kali Linux VM (Ansible orchestration & validation)
* **Target Host:** Ubuntu 26.04 Server (`castroserver`)
* **Audit Engine:** Lynis v3.1.6

```text
[Kali Linux Control Node] ──(SSH + Ansible Playbook)──> [Ubuntu 26.04 Server]
                                                               │
                                                        [Lynis Audit Engine]
                                                        Score: 56 ──> 74
```

## Audit Results (Before vs. After)

| Metric | Baseline Scan | Post-Hardening (v1.0) | Change |
| :--- | :---: | :---: | :---: |
| **Lynis Hardening Index** | **56** | **74** | **+18 (+32.1%)** |
| **Tests Performed** | 256 | 266 | +10 |
| **Intrusion Software** | None (`[X]`) | Active (`[V]`) | `fail2ban` + `auditd` deployed |
| **File Integrity (FIM)** | None (`[X]`) | Installed (`[V]`) | `aide` deployed |

## Key Security Controls Implemented
1. **Automated Patching & Vulnerability Remediation:** Upgraded vulnerable packages and configured `unattended-upgrades` and `apt-listchanges`.
2. **Intrusion Prevention & Auditing:** Installed and enabled `fail2ban` (brute-force protection), `auditd` (system call auditing), `libpam-tmpdir` (isolated `/tmp` per user), and `aide` (File Integrity Monitoring).
3. **File & Directory Permissions:** Restricted access to `/etc/sudoers.d` (`0750`), `/etc/ssh/sshd_config` (`0600`), `/etc/crontab` (`0600`), and `/etc/cron.*` directories (`0700`).
4. **SSH Daemon Hardening:** Disabled root login (`PermitRootLogin no`), X11/TCP/Agent forwarding, and reduced `MaxAuthTries` to `3` with `VERBOSE` logging and `sshd -t` config validation.
5. **Kernel (`sysctl`) Hardening:** Enforced `fs.protected_fifos=2`, `fs.suid_dumpable=0`, `kernel.kptr_restrict=2`, `kernel.sysrq=0`, BPF hardening, reverse-path filtering (`rp_filter=1`), martian packet logging, and disabled ICMP redirects.
6. **Legal Login Banners:** Configured authorized-use warning banners in `/etc/issue` and `/etc/issue.net`.

### Deliberate Operational Exceptions
* **IPv4 Forwarding (`net.ipv4.conf.all.forwarding=1`):** Intentionally preserved so active network tunneling and container services continue routing traffic without disruption.

## Troubleshooting & Lessons Learned
* **VirtualBox NAT & IPv6 Package Resolution:** Diagnosed an IPv6 `Network is unreachable` error during `apt` updates on the Kali control node and resolved it via persistent IPv4 `apt` enforcement and static NAT addressing.
* **Check-Mode Idempotency:** Added `when: not ansible_check_mode` on service startup tasks for newly installed packages (`fail2ban`, `auditd`) so `--check` dry runs complete cleanly (`failed=0`) on fresh hosts.
