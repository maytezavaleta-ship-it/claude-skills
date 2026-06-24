---
name: validate-feature-request
description: Use this skill when the user wants to validate a customer issue or feature request, check how many customers are asking for something, assess pain point severity, score a product idea against real customer signal, or phrases like "is this a real problem", "how validated is this", "do customers want this", "check VOC for X", or "validate this feature".
---

# Feature Request Validator

Validates a customer issue or feature request against real customer signal from four sources: VOC records in org 62, IdeaExchange ideas, Slack conversations, and customer meeting transcripts (Google Drive + Gong).

Produces a **0–10 validation score** where 10 = strongly validated, high pain, 0 = no signal found.

---

## Step 1 — Clarify the request

Before searching, make sure you have a clear **topic string** to search across all sources. If the user's request is vague, ask one question:

> "What's the core problem or feature in 2–5 words? (e.g. 'bulk export limits', 'SSO login failures', 'custom dashboard sharing')"

Once you have it, proceed.

---

## Step 2 — Search all sources in parallel

Run all four searches simultaneously.

---

### Source A: VOC Records (Org 62)

Use the **Bash tool** to query org 62 via curl. Auth is extracted from the browser cookie store using `pycookiecheat` — no manual steps needed as long as the user is logged into org 62 in Chrome or Arc.

```bash
python3 -c "
import json, urllib.parse
from pycookiecheat import chrome_cookies

sid = chrome_cookies('https://org62.my.salesforce.com').get('sid')
if not sid:
    # fallback to file
    try:
        sid = open('/Users/mayte.zavaleta/.voc-validator/org62_session.txt').read().strip()
    except: pass
if not sid:
    print('AUTH_FAILED'); exit(1)

sosl = 'FIND {TOPIC_KEYWORDS} IN ALL FIELDS RETURNING VOC_Product_Input__c(Name, Description__c, Business_Impact__c, Product_Area__c, Accounts_Impacted__c, Cases_Impacted__c, Total_ACV_Impacted__c, Status__c, Planning_Status__c, Total_Vote_Point_Value__c, VOC_Created_Date__c LIMIT 20)'
url = 'https://org62.my.salesforce.com/services/data/v59.0/search/?q=' + urllib.parse.quote(sosl)

import subprocess
r = subprocess.run(['curl','-s','-H',f'Authorization: Bearer {sid}', url], capture_output=True, text=True)
data = json.loads(r.stdout)
if isinstance(data, list) and data[0].get('errorCode') == 'INVALID_SESSION_ID':
    print('SESSION_EXPIRED')
    exit(1)
records = data.get('searchRecords', [])
print(f'Found {len(records)} VOC records')
for r in records:
    print(f\"\n--- {r.get('Name','')} ---\")
    print(f\"  Area: {r.get('Product_Area__c','(blank)')}\")
    print(f\"  Status: {r.get('Status__c','')} / {r.get('Planning_Status__c','')}\")
    print(f\"  Accounts impacted: {r.get('Accounts_Impacted__c','')}\")
    print(f\"  ACV impacted: {r.get('Total_ACV_Impacted__c','')}\")
    print(f\"  Vote points: {r.get('Total_Vote_Point_Value__c','')}\")
    print(f\"  Created: {r.get('VOC_Created_Date__c','')}\")
    print(f\"  Description: {str(r.get('Description__c',''))[:200]}\")
    print(f\"  Business impact: {str(r.get('Business_Impact__c',''))[:200]}\")
"
```

Replace `TOPIC_KEYWORDS` with 2–4 relevant keywords for the topic (space-separated, no quotes inside the braces).

**Note:** Run this via the Bash tool, NOT the aisuite Python sandbox (the sandbox blocks subprocess).

---

### Source B: IdeaExchange (Org 62)

Same auth pattern, same Bash tool approach. IdeaExchange lives on the same org 62 instance.

```bash
python3 -c "
import json, urllib.parse
from pycookiecheat import chrome_cookies

sid = chrome_cookies('https://org62.my.salesforce.com').get('sid')
if not sid:
    try:
        sid = open('/Users/mayte.zavaleta/.voc-validator/org62_session.txt').read().strip()
    except: pass
if not sid:
    print('AUTH_FAILED'); exit(1)

sosl = 'FIND {TOPIC_KEYWORDS} IN ALL FIELDS RETURNING Idea(Title, Body, VoteTotal, VoteScore, Status, CreatedDate, NumComments LIMIT 20)'
url = 'https://org62.my.salesforce.com/services/data/v59.0/search/?q=' + urllib.parse.quote(sosl)

import subprocess
r = subprocess.run(['curl','-s','-H',f'Authorization: Bearer {sid}', url], capture_output=True, text=True)
data = json.loads(r.stdout)
if isinstance(data, list) and data[0].get('errorCode') == 'INVALID_SESSION_ID':
    print('SESSION_EXPIRED'); exit(1)
records = data.get('searchRecords', [])
print(f'Found {len(records)} IdeaExchange ideas')
for r in records:
    print(f\"\n--- {r.get('Title','')} ---\")
    print(f\"  Status: {r.get('Status','')} | Votes: {r.get('VoteTotal','')} | Score: {r.get('VoteScore','')} | Comments: {r.get('NumComments','')}\")
    print(f\"  Categories: {r.get('Categories','')}\")
    print(f\"  Created: {r.get('CreatedDate','')}\")
    print(f\"  Body: {str(r.get('Body',''))[:200]}\")
"
```

**Note:** Run this via the Bash tool, NOT the aisuite Python sandbox.

---

### Source C: Slack

Use `mcp__plugin_slack_slack__slack_search_public_and_private` with the topic string. Run 2–3 searches with different keyword combinations.

Capture:
- Number of distinct threads mentioning the issue
- Customer names or account types mentioned
- Urgency language ("blocking", "critical", "every customer asks", "deal-breaker")
- Date range of mentions

---

### Source D: Customer Meeting Transcripts

Use `mcp__plugin_google_google__docs_search` with:
- `[topic] transcript`
- `[topic] customer meeting notes`
- `[topic] QBR`

Capture:
- Number of distinct customer meetings where topic came up
- Customer names / account sizes if visible
- Whether the customer raised it as a blocker or nice-to-have

---

## Step 3 — Score the evidence

| Dimension | Max points | How to score |
|---|---|---|
| **Breadth** — How many distinct customers/accounts mentioned this? | 3 pts | 0 = none, 1 = 1–2 accounts, 2 = 3–5 accounts, 3 = 6+ accounts |
| **Depth** — How severely is it felt? | 3 pts | 0 = no urgency language, 1 = minor friction, 2 = workaround needed, 3 = blocking/deal-breaker |
| **Recency** — How fresh is the signal? | 2 pts | 0 = >12 months ago only, 1 = 3–12 months ago, 2 = within last 3 months |
| **Source diversity** — How many of the 4 sources confirmed it? | 2 pts | 0.5 pts per source with signal (VOC, IdeaExchange, Slack, Transcripts) |

**Final score = sum of all dimensions (0–10)**

---

## Step 4 — Output format

```
## Feature Validation: [Topic]

### Score: X/10 — [Label]

[Label scale: 0-2 = No signal, 3-4 = Weak, 5-6 = Moderate, 7-8 = Strong, 9-10 = Critical]

---

### Evidence Summary

**VOC Org 62** (X records found)
- [Key record: name, accounts impacted, ACV, planning status]

**IdeaExchange** (X ideas found)
- [Top idea: title, vote count, status]

**Slack** (X mentions across Y threads)
- [Key quote or paraphrase with channel + rough date]
- Urgency markers found: [yes/no + examples]

**Customer Meeting Transcripts** (X docs found)
- [Customer/meeting context + what was said]

---

### Score Breakdown

| Dimension | Score | Notes |
|---|---|---|
| Breadth (distinct accounts) | X/3 | [brief note] |
| Depth (severity) | X/3 | [brief note] |
| Recency (freshness) | X/2 | [brief note] |
| Source diversity | X/2 | Sources with signal: [list] |
| **Total** | **X/10** | |

---

### Recommendation

[1–2 sentences: should this be prioritized? what's the strongest evidence? what's missing?]

### Gaps / Next Steps

- [e.g. "Only 1 Gong transcript found — check Gong directly"]
```

---

## Notes on auth

Auth is fully automatic — `pycookiecheat` reads the live session cookie from Chrome/Arc. You never need to provide a SID manually.

If it fails:
- Make sure you're logged into `https://org62.my.salesforce.com` in your browser, then retry
- If macOS Keychain prompts for a password, enter your Mac login password and click "Always Allow"
- Emergency fallback only: `echo 'SID_VALUE' > ~/.voc-validator/org62_session.txt`

**IMPORTANT:** Always run VOC and IdeaExchange queries via the **Bash tool**, not the aisuite Python sandbox — the sandbox blocks `subprocess` and will fail.
