# Task 3 — Remediation, Re-scan & Risk Reporting

## Goal (in one sentence)

Take real vulnerabilities you found, **fix them**, **re-scan to prove they're gone**, measure how much risk you removed, and write a short **report a non-technical manager could understand**.

> **Think of it like this:** Finding problems is only half the job. A good doctor doesn't just diagnose — they treat you, then check that you've recovered, then explain it in plain words. This task is the full cycle: **Discover → Prioritize → Remediate → Verify → Report.**

---

## What the words mean

| Term | Plain explanation |
|------|-------------------|
| **Remediation** | Fixing a vulnerability (a patch, a setting change, or hardening a service). |
| **Validation re-scan** | Running the *same scan again* after fixing, to confirm the problem is really gone. |
| **Patch** | Installing a software update that closes a known hole. |
| **Configuration change** | Adjusting a setting to a safer value. |
| **Service hardening** | Locking down a running service (disable it, restrict access, enforce encryption). |
| **Risk reduction** | How much safer the system is now, as a percentage. |
| **Executive report** | A plain-language summary for managers — business impact, not technical detail. |
| **Snapshot** | A saved "restore point" of your VM you can roll back to if a fix breaks something. |

> **The key idea:** A vulnerability isn't truly handled until you've **verified the fix** with a re-scan and **communicated the result** to decision-makers. That's vulnerability *management*, not just vulnerability *scanning*.

---

## 3.1 — Selecting vulnerabilities to remediate

From your **Task 1** results, pick **3–5** vulnerabilities to fix. Choose findings that are:

- **High or Critical** severity (biggest impact)
- **Fixable in your lab** (no production constraints, no risky kernel-level dependencies)
- A **mix of types** — at least one **patch**, one **configuration change**, and one **service hardening**

**Remediation plan**

| # | Vulnerability / CVE | Severity | Type | Planned Fix |
|---|---------------------|----------|------|-------------|
| 1 | | *(Critical/High)* | *(Patch / Config / Service)* | |
| 2 | | | | |
| 3 | | | | |
| 4 | | | | |
| 5 | | | | |

> **Why a mix of types?** Real environments need all three skills. A patch fixes outdated software; a config change fixes an unsafe setting; hardening locks down a service you can't remove. Showing one of each proves you can handle the full range.

---

## 3.2 — Applying remediations

Document the **exact steps** for each fix. Repeat the block below for each vulnerability.

> ⚠️ **Take a VM snapshot first!** Before changing anything, snapshot the VM so you can roll back instantly if a fix breaks the system. This is standard practice — never skip it.

### Remediation #1

| Field | Detail |
|-------|--------|
| Vulnerability Name / CVE | |
| Pre-Remediation State | *(Describe the issue + paste the Nessus output excerpt)* |
| Actions Taken | *(Exact commands or config changes applied)* |
| Post-Remediation State | *(Verification output after the fix)* |
| Time to Remediate | |

📸 **Screenshot required:** Terminal output showing remediation #1

### Remediation #2

| Field | Detail |
|-------|--------|
| Vulnerability Name / CVE | |
| Pre-Remediation State | *(Describe the issue + paste the Nessus output excerpt)* |
| Actions Taken | *(Exact commands or config changes applied)* |
| Post-Remediation State | *(Verification output after the fix)* |
| Time to Remediate | |

📸 **Screenshot required:** Terminal output showing remediation #2

### Remediation #3

| Field | Detail |
|-------|--------|
| Vulnerability Name / CVE | |
| Pre-Remediation State | *(Describe the issue + paste the Nessus output excerpt)* |
| Actions Taken | *(Exact commands or config changes applied)* |
| Post-Remediation State | *(Verification output after the fix)* |
| Time to Remediate | |

📸 **Screenshot required:** Terminal output showing remediation #3

> *(Add Remediation #4 and #5 blocks here if you chose 4–5 vulnerabilities.)*

> **Common fixes by type:**
> - **Patch:** `sudo apt update && sudo apt upgrade`, or upgrade the one named package.
> - **Config:** edit the service's config file (e.g. disable weak SSL/TLS in `sshd_config` or Apache), then restart the service.
> - **Service hardening:** stop/disable an unneeded service — `sudo systemctl disable --now <service>` — or restrict who can reach it.

---

## 3.3 — Validation re-scan

After **all** fixes are applied, run a **new Nessus scan with the identical settings** as the original (same template, same credentials, same ports). Only then is the comparison fair.

| Field | Value |
|-------|-------|
| Re-scan Start Time | |
| Re-scan End Time | |
| Scan Template | Same as original |
| Credentials | Same as original |

📸 **Screenshot required:** Re-scan results summary

> **Why "identical settings"?** If you change the scan, you can't tell whether a finding disappeared because you *fixed* it or because the new scan simply *didn't look* for it. Keep everything the same except the fixes.

---

## 3.4 — Risk reduction measurement

Compare the counts before and after.

| Severity | Before | After | Change | Status |
|----------|--------|-------|--------|--------|
| Critical | | | | *(Fixed / Partial / Open)* |
| High | | | | *(Fixed / Partial / Open)* |
| Medium | | | | *(Fixed / Partial / Open)* |
| Low | | | | *(Fixed / Partial / Open)* |
| **Total** | | | | *(X% reduction)* |

**Overall risk reduction:**

```
(Before − After) ÷ Before × 100 = ____ %
```

> **Example:** 20 findings before, 12 after → (20 − 12) ÷ 20 × 100 = **40% reduction**.

---

## 3.5 — Executive risk report

Write this for a **non-technical audience** (a manager, a business owner). **No jargon** — focus on *business impact* and *what it means for them*.

**1. Purpose**
> *(2–3 sentences: what you assessed and why.)*

**2. Key findings — before remediation**
> *(Summarise the vulnerabilities in plain language.)*

| Risk Level | Description | Business Impact |
|-----------|-------------|-----------------|
| Critical | *(plain explanation)* | *(e.g. full system takeover)* |
| High | *(plain explanation)* | *(e.g. customer data exposure)* |
| Medium | *(plain explanation)* | *(e.g. service outage)* |

**3. Actions taken**
> *(Describe the fixes in business-friendly terms — "applied security updates," "disabled an insecure service," not command lines.)*

**4. Current risk status**
> *(State the risk reduction in simple terms — e.g. "We removed X% of the serious risks.")*

**5. Remaining risks & recommendations**
> *(What's still open, and the suggested next steps with timelines.)*

> **Golden rule for this section:** if your grandmother couldn't understand a sentence, rewrite it. Managers care about *money, data, downtime, and reputation* — translate every technical finding into one of those.

---

## Deliverables checklist

- [ ] Remediation selection table completed
- [ ] 3–5 remediation sections completed with commands and evidence
- [ ] Validation re-scan executed
- [ ] Risk reduction table completed
- [ ] Overall risk reduction calculated
- [ ] Executive risk report completed
- [ ] Original and re-scan reports submitted

---

## Reference — CVSS severity scale

Use this to prioritise: higher severity = fix sooner.

| Severity | Score Range | Priority |
|----------|-------------|----------|
| Critical | 9.0 – 10.0 | Immediate (24–72 hours) |
| High | 7.0 – 8.9 | Urgent (within 7 days) |
| Medium | 4.0 – 6.9 | Planned (within 30 days) |
| Low | 0.1 – 3.9 | Scheduled (within 90 days) |
| Informational | 0.0 | No action required |
