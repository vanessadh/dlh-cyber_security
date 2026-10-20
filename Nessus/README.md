# Nessus Vulnerability Scanning Lab

Vulnerability assessment lab using **Nessus Essentials**, run against intentionally vulnerable and standard targets in an isolated VirtualBox network.

## Lab environment

Pre-built VirtualBox VMs on a host-only network (`192.168.56.0/24`):

| VM | Role | IP | Login |
|----|------|----|-------|
| Kali Linux | Scanner (runs Nessus Essentials 10.12.5) | 192.168.56.10 | kali / kali |
| Ubuntu Server | Target (hosts DVWA; also credentialed target) | 192.168.56.20 | user / user |
| Metasploitable 2 | Target (Task 2) | 192.168.56.101 (DHCP) | msfadmin / msfadmin |

**Networking:** the VMs must run in **VirtualBox** (not VMware) to avoid import/disk/network issues. Kali and Ubuntu share a host-only adapter so the scanner can reach the target. Confirm connectivity with `ping 192.168.56.20` from Kali before scanning.

**Scanning rule:** only scan the lab VMs — they belong to you, so no external authorisation is needed. Never scan a machine you do not own.

## Nessus setup (on Kali)

```bash
sudo apt install ./Nessus-*.deb                 # install the Debian/Ubuntu package
sudo systemctl enable --now nessusd             # start + enable the service
sudo systemctl status nessusd                   # expected: active (running)
```

Then open `https://localhost:8834`, register for **Nessus Essentials**, enter the activation code, create an admin account, and let the plugins compile (15–40 min).

Useful commands:
```bash
sudo /opt/nessus/sbin/nessuscli fetch --check                    # check license
sudo /opt/nessus/sbin/nessuscli fetch --register YOUR_CODE       # register a new code
sudo /opt/nessus/sbin/nessuscli lsuser                           # list users
```

## Tasks

| Task | Title | File | Status |
|------|-------|------|--------|
| 0 | Web Application Scan on DVWA | [task0.md](task0.md) | Complete (pending Section 5 PDF export screenshot) |
| 1 | Credentialed Scan & Local System Audit | [task1.md](task1.md) | Complete (pending 1.5 detail screenshot + report export) |
| 2 | Comparative Analysis: Nessus Basic vs Web Application Tests | [task2.md](task2.md) | Complete (pending both PDF/HTML report exports) |
| 3 | Remediation, Re-scan & Risk Reporting | [task3.md](task3.md) | Complete (pending original + re-scan report exports) |

### Task summaries

- **Task 0 — DVWA (Web Application Tests):** scanned DVWA at `http://192.168.56.20/dvwa` with an authenticated cookie. 10 findings, all Informational — a general scanner surfaces mostly info-level issues on DVWA because its deliberate vulnerabilities are application-logic flaws.
- **Task 1 — Credentialed vs Unauthenticated (Basic Network Scan):** ran the same template against `192.168.56.20` twice — once with SSH credentials, once without. Credentialed found 28 findings vs 27 unauthenticated; the credentialed scan ran SSH-only local plugins the unauthenticated one could not, which reported *"OS Security Patch Assessment Not Available."*
- **Task 2 — Basic Network Scan vs Web Application Tests:** ran both templates against Metasploitable 2 (`192.168.56.101`). The Basic scan found more and more-severe issues (9 Critical, incl. VNC password, Bind Shell backdoor, Samba Badlock); the Web App scan went deeper on the websites (CGI RFI/LFI, path traversal, XSS). Breadth vs depth — a full assessment needs both.
- **Task 3 — Remediation, Re-scan & Risk Reporting:** fixed three high-impact findings on Metasploitable (removed the Bind Shell root backdoor on port 1524, locked down the world-readable NFS export, disabled the rlogin/rsh/rexec r-services), re-scanned to confirm, and wrote an executive risk report. The three targeted findings were eliminated (68 → 62 distinct findings); the remaining criticals are the unpatchable end-of-life OS and un-targeted services.

## Folder structure

```
Nessus/
├── README.md
├── task0.md
├── task1.md
├── task2.md
├── task3.md
└── img/
    ├── task0/        # screenshots referenced by task0.md
    ├── task1/        # screenshots referenced by task1.md
    ├── task2/        # screenshots referenced by task2.md
    └── task3/        # screenshots referenced by task3.md
```
