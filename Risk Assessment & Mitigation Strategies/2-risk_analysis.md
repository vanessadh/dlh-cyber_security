# 2. Risk Analysis

**Learning Objective:** Calculate CVSS scores and ALE for vulnerabilities

---

### Scenario

A vulnerability scan reveals: **Unauthenticated remote code execution on web server**

**Given Data:**

- Asset Value: $500,000
- Exposure Factor: 80%
- ARO: 0.2

---

### Exercise

**Calculate CVSS v3.1 score and ALE:**

| CVSS Metric | Your Value | Justification |
|---|---|---|
| Attack Vector (N/A/L/P) | **N** (Network) | "Remote" means the attacker can attack from anywhere over the internet |
| Attack Complexity (L/H) | **L** (Low) | No special conditions are needed; the attack works every time |
| Privileges Required (N/L/H) | **N** (None) | "Unauthenticated" means the attacker does not need any account or login |
| User Interaction (N/R) | **N** (None) | No user needs to click or do anything; the attacker does it alone |
| Scope (U/C) | **U** (Unchanged) | The attack affects the web server itself, not other separate systems |
| Confidentiality (N/L/H) | **H** (High) | Running code on the server lets the attacker read all data on it |
| Integrity (N/L/H) | **H** (High) | The attacker can change or delete any file or data on the server |
| Availability (N/L/H) | **H** (High) | The attacker can shut down or crash the server |
| **CVSS Score** | **9.8** | Vector: `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H` |
| **Severity** | **Critical** | Scores from 9.0 to 10.0 are rated Critical |

| Financial Metric | Calculation | Result |
|---|---|---|
| SLE | Asset Value × Exposure Factor = $500,000 × 0.80 | **$400,000** |
| ALE | SLE × ARO = $400,000 × 0.2 | **$80,000** |

---

### Deliverable

**CVSS assessment:** The vulnerability scores **9.8 (Critical)**. Anyone on the internet can take full control of the web server without a password and without anyone's help.

**ALE calculation:** One attack would cost **$400,000** (SLE). The attack is expected about once every 5 years (ARO 0.2), so the average yearly loss is **$80,000** (ALE).
