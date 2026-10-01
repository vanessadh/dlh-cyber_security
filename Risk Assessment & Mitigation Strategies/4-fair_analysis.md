# 4. Risk Assessment Methodologies

**Learning Objective:** Apply FAIR methodology for quantitative risk analysis

---

### Scenario

**GlobalTech** faces a data breach risk: misconfigured cloud storage exposing 50,000 customer records.

---

### Exercise

**Complete FAIR analysis:**

| FAIR Component | Your Estimate | Reasoning |
|---|---|---|
| **Threat Event Frequency** (events/year) | **1 event/year** | Hackers use automated tools that scan the internet all day looking for open cloud storage. We expect someone to find and try to access this storage about once a year. |
| **Vulnerability** (0-1 probability) | **0.8 (80%)** | The storage is already misconfigured (open to the public), so almost nothing stops an attacker once they find it. It is not 1.0 because the attacker still has to find the exact storage name, and some access logging may catch them. |
| **Loss Event Frequency** (TEF × Vuln) | **0.8 events/year** | 1 × 0.8 = 0.8, meaning a real data breach is expected about 8 times every 10 years, which is almost every year. |

| Loss Category | Estimated Cost |
|---|---|
| Response & Investigation | **$150,000** (forensic experts, IT staff time, lawyers to investigate and fix the breach) |
| Notification costs ($5/record) | **$250,000** (50,000 records × $5) |
| Regulatory fines | **$500,000** (fines for failing to protect personal data, e.g. under GDPR) |
| Reputation damage | **$300,000** (customers leaving, lost new business, PR campaign to rebuild trust) |
| **Total Loss Magnitude** | **$1,200,000** ($150,000 + $250,000 + $500,000 + $300,000) |

| Final Calculation | Value |
|---|---|
| LEF × LM = **Annualized Risk** | 0.8 × $1,200,000 = **$960,000** |

**Risk Treatment Recommendation:** **Mitigate immediately**, then **transfer** what is left.

The expected loss is **$960,000 per year**, and the fix is cheap and fast, so GlobalTech should act right away:

1. **Block public access** on the cloud storage (this fix takes minutes and removes most of the risk).
2. **Encrypt** the customer records so stolen data is unreadable.
3. **Give access only to people and systems that need it** (least privilege).
4. **Turn on logging and alerts** to detect anyone accessing the data.
5. **Use automatic configuration checks** (a Cloud Security Posture Management tool) so the storage cannot be made public again by mistake.
6. **Buy cyber insurance** to cover the small risk that remains after the fixes.

Accepting the risk is not an option because the loss is far too high. Avoiding the risk would mean not storing customer data at all, which the business cannot do.

---

### Deliverable

**Completed FAIR analysis:** A data breach is expected **0.8 times per year**, and each breach would cost about **$1,200,000**, giving an annualized risk of **$960,000**. GlobalTech should fix the misconfiguration immediately and add cyber insurance for the remaining risk.

> **Note:** The scenario only gives the notification cost ($5/record). The other values (TEF, Vulnerability, response costs, fines, and reputation damage) are reasonable estimates, as FAIR expects the analyst to estimate them.
