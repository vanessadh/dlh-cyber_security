# Task 1 — Credentialed Scan & Local System Audit

## Goal (in one sentence)

Give Nessus a **valid login** to the target machine so it can inspect it **from the inside**, and discover the weaknesses that an outside-only scan can never see.

> **Think of it like this:** An unauthenticated scan is like a home inspector walking *around the outside* of a house, checking the doors and windows. A **credentialed scan** is like giving the inspector a **key** — now they go inside, open every cupboard, and check the wiring and plumbing too. Far more thorough.

---

## What the words mean

| Term | Plain explanation |
|------|-------------------|
| **Unauthenticated scan** | Nessus probes the target from outside, with **no login**. Sees only what's exposed to the network. |
| **Credentialed scan** | Nessus **logs in** with a real username/password (or SSH key) and checks the system from the inside. |
| **SSH** | *Secure Shell* — the standard way to log into a Linux machine remotely. Nessus uses it to get inside Linux targets. |
| **Missing patch** | A security update that hasn't been installed, leaving a known hole open. |
| **Misconfiguration** | A setting left in an unsafe state (e.g. a weak password, an open service). |
| **Plugin** | A single check Nessus runs. A scan runs thousands of them. |
| **Plugin Output** | The raw evidence Nessus collected to prove a finding — e.g. the exact version it saw. |

> **The key idea of this task:** Most real vulnerabilities (missing patches, outdated software, weak local settings) are **invisible from the network**. Only a credentialed scan finds them — which is why it's called the **gold standard** for internal systems.

---

## 1.1 — Lab environment setup

Pick **one** local VM as your target:

| Option | OS | IP | Credentials |
|--------|----|----|-------------|
| **A** (recommended) | Ubuntu Server 22.04 | `192.168.56.20` | `user` / `user` |
| **B** | Metasploitable 2 | `192.168.56.3` | `msfadmin` / `msfadmin` |

> **Authorized targets only:** Scan **only** your own lab VMs (`192.168.56.20` or `192.168.56.3`). They belong to you, so no outside permission is needed. Never point this scan at a machine you don't own — that's illegal.

**Student details**

| Field | Your Answer |
|-------|-------------|
| Selected Target | *(Ubuntu `192.168.56.20` / Metasploitable `192.168.56.3`)* |
| Target IP Address | |
| Nessus Version | *(e.g. Nessus Essentials 10.x)* |

**Pre-lab checklist — do this BEFORE scanning:**

- [ ] Nessus Essentials is installed and activated
- [ ] Target VM is powered on and reachable (ping test passes)
- [ ] You have valid credentials for the target (username & password, or SSH key)
- [ ] Nessus and the target VM are on the same network segment / host-only adapter
- [ ] You have noted the target IP address

> **Quick ping test** (run on Kali): `ping 192.168.56.20` — if you get replies, the target is reachable. Press `Ctrl+C` to stop.

📸 **Screenshot required:** Ping test result confirming the target VM is reachable

---

## 1.2 — Configure credentials in Nessus

This is the step that makes the scan "credentialed" — you tell Nessus how to log in.

**Steps**

1. In Nessus, go to **Settings → Credentials**.
2. Choose credential type: **SSH** (both targets are Linux).
3. Enter the username and password (or upload a private SSH key), then **save**.
4. When you build your scan, attach these credentials under the scan's **Credentials** tab.

📸 **Screenshot required:** Nessus credential configuration screen with type and username filled in

**Credential configuration record**

| Field | Your Entry |
|-------|-----------|
| Credential Type | SSH |
| Username | *(user / msfadmin)* |
| Authentication Method | Password |
| Target IP | |
| Credential Verification | *(Did Nessus confirm authentication? Yes / No)* |

> **Check this worked!** If authentication fails, the scan silently falls back to unauthenticated mode and you lose the whole point of the task. Nessus shows a check result or a credential-related plugin (e.g. "authentication success") if the login succeeded.

---

## 1.3 — Scan configuration & execution

Use the **Basic Network Scan** template, with your credentials attached.

📸 **Screenshot required:** Scan settings page showing target IP, template, and credentials tab

**Scan metrics**

| Metric | Value |
|--------|-------|
| Scan Start Time | |
| Scan End Time | |
| Total Duration | |
| Total Plugins Executed | |
| Hosts Scanned | |

📸 **Screenshot required:** Scan progress / completion screen

---

## 1.4 — Results analysis: credentialed vs unauthenticated

The heart of the task: **compare** what each scan type found on the **same host**.

> To fill the right column, run (or reuse) an **unauthenticated** Basic Network Scan on the same IP — same template, but with **no credentials attached**. Then compare the two result summaries side by side.

| Severity | Credentialed Scan | Unauthenticated Scan |
|----------|-------------------|----------------------|
| Critical | | |
| High | | |
| Medium | | |
| Low | | |
| Info | | |
| **TOTAL** | | |

📸 **Screenshot required:** Nessus results summary for the credentialed scan

**List 3 vulnerabilities that appeared ONLY in the credentialed scan** (not in the unauthenticated one). For each, explain **why it was hidden** from the outside scan.

1.
2.
3.

> **Hint for the "why":** These are usually **local** issues — a missing OS patch, an outdated installed package, a weak local setting. From outside, the service looked fine; only logging in revealed the out-of-date version underneath.

---

## 1.5 — Deep dive: one local vulnerability

Pick **one** finding from the **credentialed** scan (ideally a local/patch issue) and fill in every field. Click the finding in Nessus to see its full detail and the **Plugin Output** tab.

| Field | Detail |
|-------|--------|
| Plugin Name | *(e.g. Ubuntu 20.04 LTS: OpenSSL Vulnerability)* |
| CVE ID | *(CVE-XXXX-XXXXX)* |
| Severity / CVSS | *(e.g. High – 7.5)* |
| Plugin Family | *(e.g. Ubuntu Local Security Checks)* |
| Plugin Output (summary) | *(paste key lines from the Plugin Output tab)* |
| Root Cause | *(Missing patch? Misconfiguration? Weak password?)* |
| Potential Impact | *(What could an attacker do if they exploited it?)* |
| Remediation Steps | *(Exact command or config change to fix it)* |

📸 **Screenshot required:** Nessus finding detail page for your chosen vulnerability

> **Tip:** For Ubuntu patch findings, the remediation is often just `sudo apt update && sudo apt upgrade`, or upgrading the one named package. Quote the exact fix Nessus recommends.

---

## 1.6 — Reflection questions

Answer each in **full sentences**, in your own words.

**Q1.** Why should organisations always prioritise **credentialed** scanning over unauthenticated scanning for internal systems?

**Q2.** What security **risks** come from storing scan credentials inside a vulnerability scanner? How would you **reduce** those risks?

**Q3.** How would you handle a case where a credentialed scan **isn't possible** on some hosts (e.g. legacy systems, locked accounts)?

> **Starter thoughts (expand these in your own words):**
> - *Q1:* accuracy — credentialed scans see missing patches/local flaws, giving far fewer false negatives.
> - *Q2:* a stolen scanner = stolen keys to every host; mitigate with least-privilege accounts, vaults/secrets managers, key rotation, and dedicated read-only scan accounts.
> - *Q3:* fall back to unauthenticated scanning plus other controls — agent-based scanning, manual audits, network segmentation, or compensating monitoring.

---

## Deliverables checklist

- [ ] Completed task document with all placeholder fields filled in
- [ ] All required screenshots embedded and clearly labeled
- [ ] Credentialed vs unauthenticated comparison table completed
- [ ] One vulnerability deep-dive table completed
- [ ] Exported Nessus scan report (PDF or HTML) submitted separately
- [ ] All reflection questions answered in full sentences
