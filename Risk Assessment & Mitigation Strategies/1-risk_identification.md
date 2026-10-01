# 1. Risk Identification

**Learning Objective:** Identify and document assets, threats, and vulnerabilities

---

### Scenario

You are conducting a risk assessment for **SecureBank's** online banking platform serving 50,000 customers.

---

### Exercise

**Create a risk register with 5 entries:**

| ID | Asset | Threat | Vulnerability | Risk Statement |
|---|---|---|---|---|
| R1 | Customer account data (Information) | Cybercriminals using stolen passwords (Adversarial) | No multi-factor authentication (MFA) and no limit on login attempts | "The cybercriminals exploiting the missing MFA and login limits in customer account data could cause account takeover and theft of customers' money" |
| R2 | Bank employees (People) | Phishing emails from attackers (Adversarial) | Staff have no regular security awareness training | "The phishing attackers exploiting the lack of security training in bank employees could cause stolen staff passwords and a data breach" |
| R3 | Online banking web application (Software) | Human error by a developer or admin (Accidental) | Updates go live without review or testing | "The accidental configuration error exploiting the missing review and testing process in the online banking application could cause exposed customer data or a full platform outage" |
| R4 | Database server (Hardware) | Server or hard disk failure (Structural) | Only one database server, with no backup server | "The hardware failure exploiting the single point of failure in the database server could cause the banking service to stop and transactions to be lost" |
| R5 | Online banking service (Services) | Flood or power outage at the data center (Environmental) | No backup data center and the recovery plan was never tested | "The flood or power outage exploiting the lack of a tested backup site in the online banking service could cause days of downtime and loss of customer trust" |

**Asset Categories:** Information (R1), Hardware (R4), Software (R3), People (R2), Services (R5)

**Threat Categories:** Adversarial (R1, R2), Accidental (R3), Structural (R4), Environmental (R5)
