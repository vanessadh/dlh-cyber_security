# Task 2 — Comparative Analysis: Nessus Basic vs Web Application Tests

## Goal (in one sentence)

Run **two different Nessus templates** on the **same target**, then compare what each one found — to learn how your **choice of template** changes the results.

> **Think of it like this:** The same patient can be examined with an X-ray or an MRI. Each machine is built to see different things. A **Basic Network Scan** and a **Web Application Tests** scan are two different "machines" — point both at the same target and you'll get two different pictures. The lesson is knowing **which tool to reach for**.

---

## What the words mean

| Term | Plain explanation |
|------|-------------------|
| **Template** | A ready-made scan recipe in Nessus, tuned for a particular job. |
| **Basic Network Scan** | A broad, general scan — checks open ports, running services, and network-level vulnerabilities. |
| **Web Application Tests** | A focused scan — digs into websites: HTTP, forms, cookies, injection flaws. |
| **Coverage** | *How much* a scan looks at (breadth). |
| **Depth** | *How deeply* it inspects one area (detail). |
| **Unique finding** | A vulnerability that **only one** of the two scans detected. |
| **False positive** | A "finding" that turns out **not** to be a real problem. |
| **CVSSv3 score** | The 0–10 severity rating (version 3 of the CVSS standard). |

> **The key idea:** No single template sees everything. Basic = **wide but shallow**; Web App Tests = **narrow but deep** on websites. Real assessments often need **both**.

---

## 2.1 — Environment & scan configuration

Both scans use **Nessus Essentials** only — no extra tools needed.

| Field | Your Input |
|-------|-----------|
| Nessus Version | *(e.g. Nessus Essentials 10.x)* |
| Common Target | Metasploitable 2 — `192.168.56.3` |
| Scan A — Template | Basic Network Scan (unauthenticated) |
| Scan B — Template | Web Application Tests |
| Port Range (both scans) | default |

> **Tip:** Run both scans in the **same session** and **write down the start and end time** of each — you'll need the durations for the comparison. Keeping the target, port range, and conditions identical is what makes the comparison fair.

> **Authorized target only:** Metasploitable 2 is your own lab VM — safe and legal to scan. Never run these templates against a machine you don't own.

📸 **Screenshots required:** Basic Network Scan results · Web Application Tests results

---

## 2.2 — Side-by-side results comparison

Fill in the numbers from each scan's summary.

| Metric | Basic Network Scan | Web Application Tests |
|--------|--------------------|-----------------------|
| Scan Duration | | |
| Critical Findings | | |
| High Findings | | |
| Medium Findings | | |
| Low Findings | | |
| Informational | | |
| **Total Findings** | | |
| CVEs with CVSSv3 Scores | | |
| Unique Findings | | |
| False Positives | | |

> **How to count "Unique Findings":** findings that appear in *this* scan but not the other one (see the next section for the list).
> **How to judge "False Positives":** look for findings that don't actually apply to the target — verify a couple manually rather than trusting the count blindly.

---

## 2.3 — Analysis of unique findings

List up to **3 findings per scan type** that the *other* scan missed — and explain **why** it was missed.

| Template | Finding Name / CVE | Severity | Why the other scan missed it |
|----------|--------------------|----------|------------------------------|
| Basic Network Scan **only** | | | |
| Basic Network Scan **only** | | | |
| Basic Network Scan **only** | | | |
| Web Application Tests **only** | | | |
| Web Application Tests **only** | | | |
| Web Application Tests **only** | | | |

> **Hint for the "why":**
> - *Basic-only findings* are usually **network/service** issues (old SSH, open SMB, vulnerable FTP) — the Web App template doesn't test those, so it never looked.
> - *Web-App-only findings* are usually **website** issues (XSS, SQL injection, insecure cookies) — the Basic template doesn't crawl web forms deeply, so it missed them.

---

## 2.4 — Template comparison

Summarise the *character* of each template, not just the numbers.

| Criteria | Basic Network Scan | Web Application Tests | Notes |
|----------|--------------------|-----------------------|-------|
| Scan Duration | | | |
| Types of Vulnerabilities | Network / Services | Web / HTTP | |
| Plugins Used | Network, ports, services | HTTP, forms, cookies | |
| Credentials Required | No (or Yes) | No | |
| Web Analysis Depth | Low | High | |
| Total Findings | | | |
| False Positive Rate | | | |

> **Takeaway to write in your own words:** the Basic scan gives **breadth** across the whole host; the Web App scan gives **depth** on the website. Choosing the right template — or running both — is what makes an assessment accurate.

---

## Deliverables checklist

- [ ] Both scans executed against Metasploitable 2 (`192.168.56.3`)
- [ ] Results comparison table completed
- [ ] Unique findings analysis completed
- [ ] Template comparison completed
- [ ] Both Nessus reports exported (PDF / HTML)
