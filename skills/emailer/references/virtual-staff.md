# VIRTUAL STAFF — router

Identity layer for outbound email. Das Experten operates **9 active inboxes** on the `dasexperten.com` apex domain (v4.1, 2026-07-13; `.de`-Workspace decommissioned 2026-07-11). Every outbound email is signed by a virtual staff member whose name, language, and role match the inbox and the recipient.

This file is the **router**. It contains hard rules, inbox map, selection logic, and gate protocol. **Persona rosters and signature blocks live in per-inbox sub-references** — load them on demand.

---

## INBOX MAP

| Inbox | Sub-reference | Purpose | Languages |
|---|---|---|---|
| `eurasia@dasexperten.com` | `virtual-staff/eurasia.md` | Universal Russian-language hub: all CIS, all topics that don't belong to other inboxes | RU only |
| `emea@dasexperten.com` | `virtual-staff/emea.md` | Universal non-Russian hub: all English, German, Italian, Spanish, Arabic | EN, DE, IT, ES, AR |
| `asean@dasexperten.com` | `virtual-staff/asean.md` | Southeast Asia hub: VN, TH, MY, PH, ID, SG distributors and retail | EN |
| `dr.badalyan@dasexperten.com` | — (no virtual staff) | Personal address of Aram Badalyan. Signed ONLY by Aram himself, only on his explicit instruction | RU, EN |
| `sales@dasexperten.com` | `virtual-staff/sales.md` | B2B sales: distributors, wholesale, contracts | RU, EN |
| `support@dasexperten.com` | `virtual-staff/support.md` | Postsale support: returns, replacements, complex cases, retention | RU, EN, ES |
| `marketing@dasexperten.com` | `virtual-staff/marketing.md` | Bloggers/UGC collaborations, retail media, platform/CRM/marketplace managers, brand content | RU, EN, DE |
| `partnerships@dasexperten.com` | `virtual-staff/partnerships.md` | SEO/GEO outreach: link building, guest posts, PR pitches, directories, affiliate, content partnerships | RU, EN |
| `hello@dasexperten.com` | `virtual-staff/hello.md` | Warm brand front: directory/aggregator registrations, website-owner correspondence, first touch outside sales context | RU, EN |

`marketing@`, `partnerships@`, `hello@` — added 2026-07-13 by Aram's decision. ⚠️ Until the infrastructure checklist in SKILL.md Section 8 (v4.1) is complete, outbound from these three is BLOCKED — persona resolution works, sending does not.
`export@` — decommissioned with `.de`, do not use.

---

## AGENT MAILBOXES — named department heads

Five of the ten Founding Ten (`dasexperten/organizacia`, the Personnel SSOT) have provisioned mailboxes. These are **real named identities**, not gender-mirrored pool personas — one fixed name per inbox, no gender-mirror swap. Full roster, routing triggers, and signature blocks: `virtual-staff/agents.md`. Avatar hard rule (portrait required wherever the name/signature appears): `virtual-staff/agent-avatars.md`.

| Inbox | Agent | Role |
|---|---|---|
| `lauda@dasexperten.com` | Lauda Briana | Head of Commerce |
| `roberta@dasexperten.com` | Roberta Di Maria | Head of Content |
| `marika@dasexperten.com` | Marika Nowicka | Head of Brand |
| `valentina@dasexperten.com` | Valentina Korolyeva | Head of Legal |
| `justina@dasexperten.com` | Justina Timber | Head of Finance |

Prefer the agent mailbox over a functional inbox when the department head owns the thread (see `agents.md` "WHEN TO USE AGENT MAILBOXES" for the trigger-by-topic table); functional inboxes (`sales@`, `partnerships@`, `marketing@`, …) remain valid otherwise. Remaining five (`lena@`, `alexandra@`/`sasha@`, `mina@`, `zina@`, `maya@`) are **reserved, not provisioned** — route via functional inboxes until Aram assigns a mailbox.

---

## GATE PROTOCOL — when to load which sub-reference

This router stays thin by default. Sub-references load only when needed.

### Trigger rules

**Load `virtual-staff/eurasia.md` when ANY of these match:**
- Body language is Russian AND inbox is `eurasia@`
- Calling skill explicitly names a eurasia@ persona (Мария, Елена, Ирина, Татьяна, Дмитрий Ковалёв)
- Calling skill passes `inbox: eurasia` parameter
- Selection Logic Step 1 resolves to `eurasia@`
- Continuity check (Step 5) finds an existing thread signed by a eurasia@ persona

**Load `virtual-staff/emea.md` when ANY of these match:**
- Body language is non-Russian AND inbox is `emea@`
- Calling skill explicitly names an emea@ persona (Klaus, Anna Schmidt, Marco, Sofia, Ahmed, Sarah)
- Calling skill passes `inbox: emea` parameter
- Selection Logic Step 1 resolves to `emea@`
- Continuity check finds an existing thread signed by an emea@ persona

**Load `virtual-staff/marketing.md` when ANY of these match:**
- Communication target is a platform/CRM/SaaS/marketplace contact (HubSpot, Salesforce, Bitrix24, AmoCRM, Wildberries KAM, Ozon manager, TikTok Shop manager, Amazon, Shopee manager, AI service partner)
- Calling skill explicitly names a marketing@ persona (Catherine, Lukas, Дарья, Артём)
- Calling skill passes `inbox: marketing` parameter
- Body topic is platform integration / API / CRM / new service partnership
- Continuity check finds an existing thread signed by a marketing@ persona

**Load `virtual-staff/partnerships.md` when ANY of these match:**
- Communication target is a website owner / webmaster / editor / journalist / blogger-as-publisher in a link-building, guest-post, PR-pitch, directory, or affiliate context
- Calling skill is `seo-master` (outreach output routes here by default)
- Calling skill explicitly names a partnerships@ persona (Ксения, Павел, Grace, Nathan)
- Calling skill passes `inbox: partnerships` parameter
- Body topic is link exchange / guest publication / media placement / citation fix / affiliate terms / content partnership

**Load `virtual-staff/hello.md` when ANY of these match:**
- Communication is a directory/aggregator registration, confirmation, or listing update
- Communication target is a website owner in a neutral (non-link-building, non-sales) context
- First-touch email where sales@ is too commercial and partnerships@ too formal
- Calling skill explicitly names a hello@ persona (Алиса, Lily, Антон)
- Calling skill passes `inbox: hello` parameter

**Load `virtual-staff/sales.md` when ANY of these match:**
- Communication target is a B2B distributor / wholesale buyer / contract negotiation
- Calling skill is `sales-hunter` (always sales@ for cold B2B)
- Calling skill explicitly names a sales@ persona (Алексей, Анна Виноградова, James, Emma)
- Calling skill passes `inbox: sales` parameter
- Body topic is pricing / volumes / distribution terms / exclusivity / commercial offer
- Continuity check finds an existing thread signed by a sales@ persona

**Load `virtual-staff/support.md` when ANY of these match:**
- Communication target is an existing customer with a postsale issue (return, replacement, complaint resolution, warranty)
- Calling skill is `review-master` for negative reviews requiring postsale follow-up
- Calling skill explicitly names a support@ persona (Ольга, Виктор, David, Rebecca, Maria Fernández)
- Calling skill passes `inbox: support` parameter
- Body topic is return / refund / replacement / warranty / order issue
- Continuity check finds an existing thread signed by a support@ persona

**Load `virtual-staff/agents.md` when ANY of these match:**
- B2B distributor/buyer cold or deal thread owned by Commerce → `lauda@`
- SEO / guest post / editor PR / content partnership → `roberta@`
- Brand / creative collaboration / design partner → `marika@`
- Contract / NDA / compliance (`legalizer` output) → `valentina@`
- Invoice / pricing / unit economics (`invoicer` output) → `justina@`
- Calling skill explicitly names a Founding Ten agent (Lauda, Roberta, Marika, Valentina, Justina) or passes `signer_name_explicit`
- Continuity check finds an existing thread signed by one of these five agents

### Multi-inbox case

If the email touches multiple inboxes (e.g., a B2B distributor asking a postsale question), load **only the inbox that matches the dominant intent**. Do not load multiple sub-references in parallel. If genuinely ambiguous — HALT and ask Aram which inbox owns this conversation.

### Default flow does NOT load sub-references

For pure delivery operations (parameters already specified, persona already named, signature block already in body) — emailer ships the email without loading any virtual-staff sub-reference. Sub-references are needed only when **persona resolution** is required.

---

## HARD RULES — non-negotiable

These rules apply across ALL inboxes, ALL languages, ALL personas.

**Hard rule (language match):** A staff member's name must be native to the language of the email body. Russian names sign only Russian-language emails. English/German/Italian/Spanish/Arabic names sign only their respective non-Russian emails. **No mixing.** A bilingual staff (e.g. Klaus DE/EN) signs DE emails and EN emails — but never RU emails.

**Hard rule (gender mirror):** Recipient male → female sender. Recipient female → male sender. Recipient gender unknown (gender-neutral name, no profile photo, generic `info@`) → default to **female sender**. Override: only on explicit Aram instruction per-message.

**Hard rule (UGC override):** Blogger / influencer / UGC outreach is ALWAYS signed by a **female** persona, regardless of recipient gender. Female-to-female reads as peer; cross-gender DM in blogger context creates flirt/pickup risk. Override: only on explicit Aram instruction per-message.

**Hard rule (continuity):** Once a thread starts with persona X, all replies in that thread are signed by persona X. Gender mirror does NOT re-evaluate per message. Persona changes only when escalation is explicit (handoff to Aram) or when Aram instructs override.

**Hard rule (cold contact lock for Aram):** Aram NEVER signs the FIRST cold-outreach email to any contact. The General Manager does not initiate first contact — this is core B2B mechanics. CEO-level signing of a cold email signals desperation, removes negotiation leverage, and burns the escalation card before the relationship exists. Cold first contact is ALWAYS signed by a virtual staff member. Aram enters later, with authority intact.

**Hard rule (customer support):** Aram NEVER signs customer-support email personally. Use the appropriate inbox roster.

**Hard rule (no fabrication):** No fabricated bank details, IBANs, SWIFT codes, tax IDs, contract numbers. If data missing, return error. Pull from `contacts` skill. Do not invent.

**Hard rule (Germany silence):** No mention of German origin, German production, "made in Germany", "немецкое производство", "German engineering" in any body text. The German-style brand identity (name, design language) is the working compromise — do not amplify in text; `.de` domain decommissioned, apex is `dasexperten.com`. Applies to ALL languages, ALL personas, ALL inboxes.

**Hard rule (single-language pool exception to gender mirror):** When a body language has only one staff member in the resolved inbox (e.g., Italian only Marco in emea@, Spanish only Sofia in emea@, Arabic only Ahmed in emea@), **language match takes priority over gender mirror**. The single-language staff signs both male and female recipients. Do NOT fall back to English just to flip gender.

---

## SELECTION LOGIC

When a calling skill needs a signer, decide in this order:

### Step 0 — Is this a COLD FIRST CONTACT?
- Recipient has never replied to Das Experten
- No prior `thread_id` exists in this conversation
- Calling skill is `sales-hunter` (cold by definition)
- Calling skill is `ugc-master` first-touch
- Or any context where "first introduction" applies
- **If yes → Aram is FORBIDDEN. Pick a virtual staff persona per Steps 1–4.**
- If unsure whether cold or warm — treat as cold. Default safe.

### Step 1 — Determine inbox by purpose
- Customer service / general inquiry / quality / order question → **eurasia@** (RU) or **emea@** (other languages)
- Platform / CRM / SaaS / marketplace manager communication, blogger/UGC collaborations, retail media → **marketing@**
- SEO/GEO outreach: link building, guest posts, PR pitches, directories, affiliate → **partnerships@**
- Directory registrations, website-owner correspondence, neutral warm first touch → **hello@**
- B2B sales / distributor / wholesale / contract negotiation → **sales@**
- Postsale / return / replacement / complex case / retention → **support@**

### Step 2 — Determine body language
- Russian → eurasia@, OR sales@ Russian sub-pool, OR marketing@ Russian sub-pool, OR support@ Russian sub-pool
- English → emea@, OR sales@ English sub-pool, OR marketing@ English sub-pool, OR support@ English sub-pool
- German → emea@ DE-capable pool, OR marketing@ DE-capable pool
- Italian → emea@ → Marco
- Spanish → emea@ → Sofia (general) OR support@ → Maria (postsale)
- Arabic → emea@ → Ahmed

### Step 3 — Load sub-reference
- Use Gate Protocol triggers above
- Load only the one sub-reference that matches resolved inbox

### Step 4 — Apply gender mirror within the language pool (per loaded sub-reference)
- Recipient male → pick female persona
- Recipient female → pick male persona
- Recipient gender unknown → default female
- Single-language pool exception applies (see Hard Rules)

### Step 5 — Apply UGC override (if applicable)
- If `is_ugc = true` → ALWAYS female, override gender mirror
- Blogger barter / UGC collaboration (ugc-master) → load `marketing.md`, female persona, from marketing@
- Legacy fallback if marketing@ still blocked: CIS UGC → `eurasia.md` → Ирина Величко; International UGC → `emea.md` → Sarah Mitchell (EN) or Sofia García (ES)

### Step 6 — Apply continuity check
- If existing `thread_id` → look up the original signer in the thread → **continue with same persona** regardless of current step output
- If new thread → use the persona resolved by Steps 1–4
- Continuity may require loading a different sub-reference than Step 3 — do so

### Step 7 — Confirm to Aram
- Display resolved persona in the confirmation gate so Aram sees who's signing
- If override needed (rare) — Aram can specify alternate signer per-message

---

## ARAM BADALYAN — escalation only

**Role:** General Manager, Das Experten International LLC / General Director, Das Experten Eurasia LLC

**Languages:** Russian, English, Armenian

**Tone:** Stoic-logical, dry, business; no corporate filler; short direct sentences

**Use for:**
- Bank correspondence (after relationship established)
- Legal counterparties (legalizer outputs) **after relationship established**
- C-level negotiations — only after counterparty has replied
- Top-tier partner escalations
- Manufacturing ownership matters with Honghui / Yangzhou
- Final commercial terms requiring GM authority (exclusivity, contract red lines)

**NEVER use for cold first contact.** Not for distributors, not for buyers, not for "important enough" prospects. The GM does not knock on doors — a virtual staff persona opens the door, Aram walks through it later. If the calling skill argues "but this is a really senior contact" — that is exactly when the rule matters most. Decline and route to the appropriate inbox + persona.

**Escalation timing — when Aram CAN enter the thread:**
- Counterparty has replied at least once and the conversation has substance (not auto-reply, not thank-you)
- A specific decision requires GM authority (final pricing, exclusivity, contract red lines)
- Counterparty asks to speak with the principal directly
- Virtual staff persona explicitly hands off ("CC'ing our GM Aram Badalyan to confirm the commercial terms")

**Aram signature, English:**
```
Aram Badalyan
General Manager
Das Experten International LLC
```

**Aram signature, Russian:**
```
Арам Бадалян
Генеральный директор
Das Experten Eurasia LLC
```

---

## CROSS-SKILL CONTRACT

A calling skill that produces an email body should pass to emailer:

- `inbox` — one of: `eurasia` | `emea` | `asean` | `marketing` | `sales` | `support` | `partnerships` | `hello`
- `language` — `ru` | `en` | `de` | `it` | `es` | `ar`
- `recipient_gender` — `male` | `female` | `unknown` (drives gender mirror)
- `is_cold` — boolean (drives cold-contact lock)
- `is_ugc` — boolean (drives UGC override)
- `stage` — `cold` | `active` | `negotiation` | `onboarding` (drives title-by-stage logic in sales@)
- OR `signer_name_explicit` — when Aram named a specific persona

emailer's job:
1. Resolve inbox → load matching sub-reference
2. Resolve persona using Selection Logic in this router + roster in sub-reference
3. Apply gender mirror within the language pool
4. Apply title-by-stage logic for sales@ personas (sub-reference handles this)
5. Confirm the body matches that persona's tone (sanity check, not full rewrite)
6. Append the canonical signature block from the sub-reference
7. Set the `from` field on the Resend payload to match the inbox
8. Display the resolved persona in the confirmation gate

**Calling skills MUST NOT pick signers themselves.** Sales-hunter, ugc-master, review-master, etc. pass the context — emailer + this router + sub-reference resolve the persona.

---

## NAME UNIQUENESS — global registry

No two personas share a first name across the ecosystem (this skill + `das-presenter/references/virtual-staff.md`). Verify before adding any new persona.

Cross-language collision pairs explicitly allowed (different language lines, never collide in single email):
- **Anna Schmidt** (DE/EN, emea@) vs **Анна Виноградова** (RU, sales@) — Latin vs Cyrillic
- **Maria Fernández** (ES/EN, support@) vs **Мария Косарева** (RU, eurasia@) — Latin vs Cyrillic

If adding new persona — load all 8 sub-references and the das-presenter virtual-staff to verify uniqueness before assignment.

---

## CHANGE LOG

- **2026-07-16 (v4.2)** — +AGENT MAILBOXES section and gate trigger for the 5 provisioned Founding Ten mailboxes (`lauda@`, `roberta@`, `marika@`, `valentina@`, `justina@`), sourced from `dasexperten/organizacia` (Personnel SSOT). New sub-references: `virtual-staff/agents.md`, `virtual-staff/agent-avatars.md`. These mailboxes previously existed only in `organizacia`'s copy of this skill and were not wired into ALLOWED_FROM (see `SKILL.md` §2.2) — brought into sync across all skill mirrors.
- **2026-07-13 (v4.1)** — +3 inboxes: marketing@ reactivated on own address, partnerships@ (SEO/GEO outreach, default for seo-master), hello@ (warm brand front). Gate triggers and Selection Logic updated; UGC override re-routed to marketing@ with legacy fallback; cross-skill inbox enum extended (asean added — was missing); emailer-bridge reference replaced with Resend. Sub-reference count: 8.

- **2026-05-03** — Restructure to 5-inbox + lazy-load architecture. This file becomes the router; rosters and signature blocks moved to per-inbox sub-references in `virtual-staff/`. Gate protocol added. 24 staff distributed across 5 inboxes (eurasia 5, emea 6, marketing 4, sales 4, support 5). Old language-block architecture replaced. New rules: language-name match, inbox-purpose routing, stage-based title shift, single-language pool exception. `export@` deprecated for outbound.
- **2026-05-02** — language-block + gender-mirror architecture (superseded).
- **2026-05-01** — initial reference scaffold.
