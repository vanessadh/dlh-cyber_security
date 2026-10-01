# Task 0 – Risk Fundamentals (TechCorp)

## The Scenario in Plain Words

TechCorp has a **customer database** worth **$2,000,000**. That database has a **SQL injection** weakness: an attacker can type special commands into a website form (like a login or search box) and trick the database into giving away or changing data.

We need to figure out:
- How likely an attack is
- How much damage one attack would cause
- How much money this risk costs per year on average
- What TechCorp should do about it

---

## Completed Risk Assessment Table

| Component | Answer |
|---|---|
| **Threat** | An attacker (hacker, criminal group, or malicious insider) who sends harmful SQL commands through the website to steal, change, or delete customer records |
| **Vulnerability** | The application does not properly check or clean user input before sending it to the database (no parameterized queries / prepared statements) |
| **Likelihood** | **Medium** |
| **Impact** | **High** |
| **Risk Level** | **High** (Medium Likelihood × High Impact) |
| **SLE (Asset × EF)** | **$800,000** |
| **ALE (SLE × ARO)** | **$240,000** |
| **Treatment** | **Mitigate** (fix the vulnerability) |

---

## Calculations Shown

### 1. SLE – Single Loss Expectancy
*"How much money do we lose if the attack happens **one time**?"*

```
SLE = Asset Value × Exposure Factor
SLE = $2,000,000 × 40%
SLE = $2,000,000 × 0.40
SLE = $800,000
```

➡️ One successful attack would cost TechCorp about **$800,000**.

The **Exposure Factor (40%)** means one attack would damage or expose 40% of the asset's value, not all of it.

### 2. ALE – Annualized Loss Expectancy
*"On average, how much money do we lose **per year** from this risk?"*

```
ALE = SLE × ARO
ALE = $800,000 × 0.3
ALE = $240,000
```

➡️ This risk costs TechCorp about **$240,000 per year** on average.

The **ARO (0.3)** means the attack is expected about **3 times every 10 years**, or roughly **once every 3–4 years**.

---

## Why These Ratings?

### Likelihood = Medium
- An ARO of 0.3 means it is **not expected every year**, but it is **realistic** within a few years.
- SQL injection is a **well-known** attack (listed in the OWASP Top 10 under *Injection*), and free tools exist to find it automatically. So it's not "Low".
- But it isn't happening multiple times a year (ARO ≥ 1), so it's not "High".

### Impact = High
- One attack = **$800,000** loss.
- Customer data leaks also bring **hidden costs**: GDPR fines (up to 4% of yearly global turnover), lawyers, notifying customers, and **lost trust** / lost clients.

### Risk Level = High
Using the risk matrix:

| | Low Impact | Medium Impact | High Impact |
|---|---|---|---|
| **High Likelihood** | Medium | High | Critical |
| **Medium Likelihood** | Low | Medium | **➡️ High** |
| **Low Likelihood** | Low | Low | Medium |

Medium Likelihood + High Impact = **High risk**.

---

## Treatment = Mitigate

**What it means:** Reduce the risk by fixing the problem.

**Why not the other options?**

| Option | Why not chosen |
|---|---|
| **Accept** | $240,000/year is too expensive to ignore, and a data breach could break the law (GDPR). |
| **Avoid** | Avoiding means shutting down the customer database or website. The business needs it, so that's not realistic. |
| **Transfer** | Cyber insurance can cover some money, but it **does not fix the hole**, does not stop fines, and does not restore customer trust. Useful only as a **backup** after fixing. |
| **Mitigate** ✅ | The fix is cheap compared to the loss, and it removes the root cause. |

**How to mitigate (simple actions):**
1. **Use parameterized queries / prepared statements** – the main fix. The database treats user input as plain text, never as commands.
2. **Validate input** – only accept what is expected (e.g., numbers only in a "phone" field).
3. **Least privilege** – the website's database account should only have the permissions it really needs (no `DROP TABLE`, no admin rights).
4. **Web Application Firewall (WAF)** – blocks common SQL injection attempts.
5. **Encrypt sensitive data** – even if stolen, it is harder to use.
6. **Regular testing** – code reviews, vulnerability scans, and penetration tests.

**Is it worth it?** If these fixes cost, for example, $50,000 and cut the yearly loss from $240,000 to around $20,000, TechCorp saves about **$170,000 per year**.

```
Value of control = ALE before − ALE after − yearly cost of control
                 = $240,000 − $20,000 − $50,000
                 = $170,000 saved per year
```
*(Example numbers to show the reasoning, not given in the task.)*

---

## Summary in One Sentence

The SQL injection flaw is a **High risk** that could cost **$800,000 per attack** and **$240,000 per year on average**, so TechCorp should **mitigate** it right away by fixing the code (prepared statements), and can add cyber insurance as an extra safety net.
