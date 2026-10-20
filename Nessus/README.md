# Nessus Vulnerability Scanning Lab

Vulnerability assessment lab using **Nessus Essentials**, run against intentionally vulnerable targets in an isolated VirtualBox network.

## Lab environment

The lab uses pre-built VirtualBox VMs on a host-only network (`192.168.56.0/24`):

| VM | Role | IP | Login |
|----|------|----|-------|
| Kali Linux | Scanner (runs Nessus Essentials) | 192.168.56.10 | kali / kali |
| Ubuntu Server | Target (hosts DVWA) | 192.168.56.20 | user / user |
| Metasploitable 2 | Target (later tasks) | 192.168.56.x | msfadmin / msfadmin |

**Networking:** the VMs must run in **VirtualBox** (not VMware) to avoid import/disk/network issues. Kali and Ubuntu share a host-only adapter so the scanner can reach the target. Confirm connectivity with `ping 192.168.56.20` from Kali before scanning.

**Scanning rule:** only scan the lab VMs — they belong to you, so no external authorisation is needed. Never scan a machine you do not own.

## Nessus setup (on Kali)

```bash
# install the Debian/Ubuntu package
sudo apt install ./Nessus-*.deb

# start and enable the service
sudo systemctl enable --now nessusd
sudo systemctl status nessusd        # expected: active (running)
```

Then open `https://localhost:8834`, register for **Nessus Essentials**, enter the activation code, create an admin account, and let the plugins compile (15-40 min).

Useful commands:
```bash
sudo /opt/nessus/sbin/nessuscli fetch --check                    # check license
sudo /opt/nessus/sbin/nessuscli fetch --register YOUR_CODE       # register a new code
sudo /opt/nessus/sbin/nessuscli lsuser                           # list users
```

## Tasks

| Task | Title | File |
|------|-------|------|
| 0 | Web Application Scan on DVWA | [task0.md](task0.md) |
| 1 | Credentialed Scan & Local System Audit | task1.md |
| 2 | Comparative Analysis: Nessus Basic vs Web Application Tests | task2.md |
| 3 | Remediation, Re-scan & Risk Reporting | task3.md |

## Folder structure

```
Nessus/
├── README.md
├── task0.md
└── img/              # screenshots referenced by task0.md
```
