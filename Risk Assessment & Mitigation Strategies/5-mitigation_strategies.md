# 5. Mitigation Strategies

**Learning Objective:** Design defense-in-depth security controls

---

### Scenario

**SecureBank** must protect its online banking platform from SQL injection attacks.

**Budget:** $100,000 | **Timeline:** 90 days

---

### Exercise

**Design a defense-in-depth strategy:**

| Layer | Control | Type | Cost | Priority |
|---|---|---|---|---|
| **Network** | **Web Application Firewall (WAF)** with SQL injection rules that block attack attempts before they reach the website | Tech | $20,000 | **High** (fastest protection while the code is being fixed) |
| **Host** | **Database server hardening**: the website's database account only gets the permissions it needs (read/write its own tables, no admin rights, no deleting tables) and the server is fully patched | Tech | $10,000 | **High** (limits the damage if an attack gets through) |
| **Application** | **Fix the code with parameterized queries (prepared statements)** and **input validation**, plus automated security testing tools (SAST/DAST) to find any remaining flaws | Tech | $40,000 | **Critical** (fixes the root cause of SQL injection) |
| **Data** | **Encrypt sensitive customer data** (account numbers, personal details) and **monitor database activity** to alert on unusual queries | Tech | $15,000 | **Medium** (stolen data is unreadable, and attacks are detected) |
| **Administrative** | **Secure coding training** for all developers and a **mandatory code review policy** before any change goes live | Admin | $10,000 | **Medium** (stops new SQL injection flaws from being written) |
| **TOTAL** | | | **$95,000** | ✅ Within the $100,000 budget ($5,000 kept as a reserve for unexpected costs) |

**Implementation Timeline:**

| Week | Action |
|---|---|
| 1-2 | **Quick protection:** deploy the WAF in blocking mode, scan the website to find all vulnerable pages, and remove extra permissions from the website's database account |
| 3-6 | **Fix the root cause:** rewrite all vulnerable database queries using prepared statements, add input validation, harden and patch the database server, and start developer training |
| 7-12 | **Strengthen and verify:** encrypt sensitive data, turn on database activity monitoring, add SAST/DAST tools to the development process, enforce the code review policy, and run a final penetration test to confirm the fixes work |

**Success Metric:** By day 90, an **independent penetration test and vulnerability scan find 0 SQL injection vulnerabilities**. In addition, **100% of database queries use prepared statements**, **100% of developers have completed secure coding training**, and the WAF and database monitoring are **actively blocking and alerting** on attack attempts.

---

### Deliverable

**Defense-in-depth plan within budget:** SecureBank protects its online banking platform with **5 layers of security**. If one layer fails, the next one still protects the data:

1. The **WAF** blocks most attacks at the door.
2. The **fixed code** makes SQL injection impossible even if an attack gets past the WAF.
3. The **limited database permissions** reduce the damage an attacker can do.
4. **Encryption** makes stolen data useless, and **monitoring** raises an alarm.
5. **Training and code reviews** stop new flaws from being created.

The total cost is **$95,000**, which is within the **$100,000 budget**, and everything is completed within the **90-day timeline**.
