# Task 0 — Web Application Scan on DVWA

## Goal (in one sentence)

Use **Nessus Essentials** to scan a deliberately broken website (**DVWA**) in your lab, find its weaknesses, and write down what you found.

> **Think of it like this:** DVWA is a practice dummy full of known problems. Nessus is an automatic security inspector. You point the inspector at the dummy, it checks hundreds of things, and it hands you a report of everything wrong. Your job is to run it and understand the report.

---

## What the words mean

| Term | Plain explanation |
|------|-------------------|
| **DVWA** | *Damn Vulnerable Web Application* — a website made **on purpose** to be insecure, so you can practise safely and legally. |
| **Nessus Essentials** | A free vulnerability scanner. It probes a target and lists known security problems, ranked by how dangerous they are. |
| **Web Application Tests** | The Nessus "template" (a ready-made recipe) tuned to test websites specifically. |
| **Vulnerability** | A weakness an attacker could use to break in or steal data. |
| **Severity** | How serious a weakness is: Critical > High > Medium > Low > Informational. |
| **CVE** | A global ID number for a known vulnerability (e.g. `CVE-2021-1234`), so everyone refers to the same bug. |
| **CVSS Score** | A number from 0 to 10 rating how bad a vulnerability is. Higher = worse. |

---

## Lab setup

| Field | Value |
|-------|-------|
| Scanner | Nessus Essentials on **Kali Linux** (`192.168.56.10`) |
| Target | DVWA on **Ubuntu Server** (`192.168.56.20`) |
| Target URL | `http://192.168.56.20/dvwa` |
| Nessus Template | Web Application Tests |

> The scanner and the target are **two separate machines** on the same private lab network (`192.168.56.x`). The Kali box does the scanning; the Ubuntu box hosts the vulnerable site.

---

## Section 1 — Prepare DVWA

Before scanning, get the target ready so the scanner can reach every part of it.

**Steps**

1. Open the web browser on **Kali Linux**.
2. Go to the DVWA URL: `http://192.168.56.20/dvwa`
3. Log in to DVWA (default credentials are usually `admin` / `password`).
4. If prompted, **initialize or reset the DVWA database** (button at the bottom of the setup page).
5. Log in again after the reset.
6. Set the **DVWA security level to `Low`** (menu: *DVWA Security*).

> **Why security level "Low"?** DVWA can hide its own weaknesses on higher settings. "Low" leaves them exposed so the scanner can actually find them — which is the whole point of the exercise.

📸 **Screenshot required:** DVWA dashboard after login

---

## Section 2 — Configure the scan

Now set up the scan inside Nessus.

**Steps**

1. Open the Nessus web interface (usually `https://127.0.0.1:8834` on Kali).
2. Create a **New Scan**.
3. Choose the **Web Application Tests** template.
4. Fill in the scan settings (name, target URL, etc.).
5. **Save** the scan configuration.

📸 **Screenshot required:** Scan configuration page

**Record your settings:**

| Setting | Value |
|---------|-------|
| Scan Name | |
| Target URL | |
| Scan Type | |
| Port Range | |

---

## Section 3 — Run the scan

**Steps**

1. **Launch** the scan.
2. **Watch** the progress bar.
3. **Wait** until it reaches 100% / "Completed".

> This can take a while. Let it finish — a half-done scan gives an incomplete report.

📸 **Screenshots required:** Scan running status · Scan completed status

**Record the timing:**

| Metric | Value |
|--------|-------|
| Start Time | |
| End Time | |
| Duration | |
| Status | |

---

## Section 4 — Analyse the results

This is the most important part: reading and understanding what Nessus found.

### 4.1 Vulnerability overview

Count how many problems Nessus found at each severity level.

| Severity | Count |
|----------|-------|
| Critical | |
| High | |
| Medium | |
| Low | |
| Informational | |
| **Total** | |

📸 **Screenshot required:** Nessus severity summary

### 4.2 Critical and High vulnerabilities

List the most serious findings — these are the ones you'd fix first in real life.

| # | Vulnerability Name | Severity | CVE | Service |
|---|--------------------|----------|-----|---------|
| 1 | | | | |
| 2 | | | | |
| 3 | | | | |
| 4 | | | | |
| 5 | | | | |

📸 **Screenshot required:** Filtered vulnerability list (Critical / High)

### 4.3 Deep dive — pick ONE vulnerability

Choose one finding (ideally a Critical or High) and study it in full. Click it in Nessus to see all its details.

| Field | Detail |
|-------|--------|
| Plugin Name | |
| CVE ID | |
| Plugin ID | |
| Plugin Family | |
| Severity | |
| CVSS Score | |
| Affected Host | |
| Port / Service | |
| Synopsis | |
| Description | |
| Impact | |
| Remediation | |
| References | |

> **Quick guide to the fields:**
> - **Synopsis** = the one-line summary.
> - **Description** = what the weakness is.
> - **Impact** = what an attacker could do with it.
> - **Remediation** = how to fix it.

📸 **Screenshot required:** Vulnerability detail page

---

## Section 5 — Export the report

Save a shareable copy of your findings.

**Steps**

1. Open the scan results.
2. Choose the **Export / Report** option.
3. Select **PDF** format.
4. Generate and **download** the report.

📸 **Screenshot required:** Report export screen

---

## Submission checklist

- [ ] DVWA prepared and accessible
- [ ] Scan configured
- [ ] Scan executed successfully
- [ ] Scan monitoring data completed
- [ ] Vulnerability summary filled
- [ ] Critical / High table completed
- [ ] Deep dive section completed
- [ ] Nessus report exported (PDF)
- [ ] All required screenshots included

---

> **Note:** DVWA installation is documented separately for builders — see *Lab environment § DVWA on Ubuntu* (one-time setup).
