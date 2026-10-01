# 6. Risk Monitoring and Review

**Learning Objective:** Design a risk monitoring dashboard

---

### Scenario

**SecureBank** needs an executive risk dashboard for quarterly board meetings.

---

### Exercise

**Design a one-page executive dashboard:**

## SecureBank – Executive Risk Dashboard (Q3 2026)

**Section 1: Risk Summary**

| Metric | Value |
|---|---|
| Total Risks | **20** |
| Critical | **2** 🔴 |
| High | **5** 🟠 |
| Medium | **8** 🟡 |
| Low | **5** 🟢 |

**Section 2: Top 3 Risks**

| Rank | Risk | Level | Owner | Status |
|---|---|---|---|---|
| 1 | SQL injection in the online banking platform (customer data could be stolen) | **Critical** 🔴 | Head of Application Development | **In progress**: WAF installed, code fix 60% complete |
| 2 | Remote code execution on the web server (attacker could take full control) | **Critical** 🔴 | IT Infrastructure Manager | **In progress**: security patch scheduled for next month |
| 3 | Customer account takeover (no multi-factor authentication) | **High** 🟠 | Head of Digital Banking | **Planned**: MFA project approved, starts next quarter |

**Section 3: KPIs**

| KPI | Target | Actual | Trend |
|---|---|---|---|
| Avg mitigation time | ≤ 30 days | 42 days | **↓** Improving (was 55 days last quarter) |
| Control effectiveness | ≥ 90% | 85% | **↑** Improving (was 78% last quarter) |
| Open action items | ≤ 10 | 14 | **→** No change (was 14 last quarter) |

**Section 4: Quarterly Actions**

- **Action 1:** Finish fixing the SQL injection code with prepared statements and confirm it with an independent penetration test by the end of the quarter.
- **Action 2:** Install the security patch on the web server to remove the remote code execution risk within 30 days.
- **Action 3:** Start rolling out multi-factor authentication (MFA) for all 50,000 customers to stop account takeovers.

---

### Deliverable

**Completed dashboard:** SecureBank currently tracks **20 risks**, including **2 Critical** and **5 High**. The two Critical risks (SQL injection and remote code execution) are both being fixed, and account takeover is next. Mitigation is getting faster (55 → 42 days) and controls are working better (78% → 85%), but both are still below target, and open action items have not gone down. The 3 actions for this quarter focus on closing the biggest risks first.

> **Note:** The scenario gives no real data, so the numbers above are realistic example values. The top 3 risks match the risks analyzed in the earlier tasks.
