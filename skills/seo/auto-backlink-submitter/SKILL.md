# Auto Backlink Submitter

## Purpose

Create legitimate off-site backlinks and brand citations through **fully automatic submission** —
open write APIs and no-captcha / no-email-verify endpoints — instead of manual form-filling or email
outreach. A GEO/off-page supporting skill: it grows referring domains and entity citations that both
classic search and AI answer engines weight, with zero human clicks per link.

## When to Use

Use this skill when the task is to:

- build foundational backlinks / brand citations at scale without manual submission;
- add a brand's products to open product/ingredient databases (e.g. Open Beauty Facts);
- discover which directories, knowledge bases, or APIs accept programmatic submission;
- support a domain-authority / GEO growth plan's link-acquisition phase.

## When Not to Use

Do not use this skill when:

- the target requires captcha, email verification, or manual editorial review (route to the
  semi-manual submission flow or to outreach instead — see `sales/` outreach skills);
- the only path is buying links or posting to link farms / PBNs — **forbidden**, it triggers
  toxic-link penalties and defeats the goal;
- the platform's ToS prohibits automated/bot submissions.

## Inputs

Brand name, site URL, canonical product URLs, EAN/GTIN barcodes, full INCI/spec per SKU, entity IDs
(e.g. Wikidata QID), and any existing platform credentials from the estate secret store.

## Core Principle

**Only legitimate, real records.** Every submission must be a genuine, accurate entry (real product,
real company) on a platform designed to accept it. Authority + truthfulness beats volume — one real
Open Beauty Facts product page outranks fifty spun directory listings and carries no penalty risk.

## Multilingual by default — cover every market, not just English

Discovery MUST run across **every language/market the site serves**, not English-only. Global open
databases (Open Beauty Facts, Wikidata) accept localized fields — add native product names,
descriptions, and language labels per locale. On top of that, each market has its **own**
auto-submittable, high-authority local platforms (national barcode/product/ingredient DBs, open local
business registries, country wikis) that a global sweep misses. A citation from a local-language,
country-relevant source is worth more in that market's SERP and AI answers than another English one.

- Run one discovery pass **per language market** the site targets (e.g. en, de, ru, es, fr, pl, tr,
  uk, ar, zh, vi, th, tl, ms — mirror the site's actual locale set).
- Fill localized fields on global platforms (multilingual labels/descriptions feed non-English
  engines and the Knowledge Graph).
- Record each created link with its language/market so the referring-domain tracker shows market
  spread, not just an English cluster.
- Respect per-region infra rules (e.g. a locale whose DNS/hosting lives outside the main provider is
  still fine to cite from local directories — the rule is about not touching that zone's DNS, not
  about avoiding its language's link sources).

## Workflow

1. **Inventory** the brand assets (site, product URLs, barcodes, INCI, entity IDs) and pull any
   existing platform accounts from secrets.
2. **Discover** auto-submittable channels — run the discovery fan-out (see *Discovery pattern*).
   Classify each: `yes-now` (scriptable today, no captcha/verify), `yes-with-account` (needs a
   scriptable login/API key you can set up), `no` (captcha/verify/manual → hand off).
3. **Execute** every `yes-now` channel with a scripted request. Register an account first where the
   write API requires credentials (store them in the secret store, never in memory).
4. **Verify** each link is live: fetch the created record's public URL and confirm the outbound link
   to the brand site is present.
5. **Log** every created link (platform, URL, link target, date) into the campaign's referring-domain
   tracker. Store new credentials in the estate secret store.
6. **Hand off** `no` channels with paste-ready copy for the semi-manual flow.

## Discovery pattern (multi-agent)

Fan out researchers on **two axes** — channel class × language market — each verifying against the
real site/API docs (not memory): open product/ingredient/barcode DBs · editable wikis / structured-
data KBs · no-captcha directories · API-key platforms · URL-submission / feed endpoints ·
profile/identity APIs — repeated for every locale the site serves (surface each market's *local*
platforms, not just the global English ones). Each returns: platform, language/market, exact endpoint
+ method, auth model, captcha?, email-verify?, `automatable` rating, what link is published, and a
ready request. Execute the `yes-now` set; queue `yes-with-account`.

## Proven channel — Open Beauty Facts (write API, account required, no captcha)

Cosmetics/oral-care open database; each product page carries an outbound `link` to the brand site,
plus an auto-generated `/brand/<slug>` facet page. Anonymous writes are rejected; a free account
(created via `cgi/user.pl`) is enough — no captcha, no email gate on the write path.

```bash
# Add / edit a product (repeat per SKU)
curl -s -X POST "https://world.openbeautyfacts.org/cgi/product_jqm2.pl" \
  --data-urlencode "code=<EAN13>" \
  --data-urlencode "user_id=<account>"  --data-urlencode "password=<password>" \
  --data-urlencode "product_name_en=<Brand Product — type>" \
  --data-urlencode "brands=<Brand>"  --data-urlencode "categories=Toothpastes" \
  --data-urlencode "quantity=<size>" \
  --data-urlencode "link=https://<brand-site>/product/<slug>" \
  --data-urlencode "ingredients_text_en=<full INCI>" \
  -H "User-Agent: <Brand>-DataSubmission/1.0 (<email>)"
# success: {"status":1,"status_verbose":"fields saved"}
```

The same Open Food Facts account works across the OFF family; use the correct sibling for the product
type (beauty/cosmetics → Open Beauty Facts). Verify: `GET /api/v2/product/<EAN>.json` → check `link`.

## Channel catalogue (from the 2026-07 auto-submit scan)

**Backlink / citation channels (create a public link):**
- **Open Beauty Facts** — `yes-now` (free account, no captcha). Product page per EAN with a `link` to
  the brand site + brand facet page. *Proven — see method above.* Cosmetics/oral-care only; use the
  correct OFF-family sibling per product type.
- **Wikidata** (`wbeditentity`/`wbsetclaim`/`wbsetreference`) — `yes-with-account`. Highest durability
  (feeds Knowledge Graph + LLMs). The item may already exist; add `official website` (P856),
  multilingual labels/descriptions, product statements — each sourced. Needs a logged-in account /
  bot password (account creation is captcha-gated → one-time human step).
- **Wikimedia Commons** (MediaWiki `action=upload`/`edit`) — `yes-with-account`. Product/logo image
  with a description linking the site; same Wikimedia login as Wikidata.
- **Code-forge profiles** (GitHub / GitLab / Codeberg) — `yes-with-account`, no captcha via API once
  the account exists. Org/user profile + profile-README with a website link (often nofollow, still a
  high-authority citation).
- **Generic barcode DBs** (UPCitemdb / upcdatabase.org) — `yes-with-account`, API-key. Product record
  with a brand URL; lower authority, mark link type on verify.

**Open-science / DOI repositories (highest legitimacy for a science-based brand)** — `yes-with-account`,
OAuth/API token, **no captcha on the API**, no editorial review on most → instant. Deposit a technical
note, dataset, protocol, or figure set (real data only) and the publish step mints a **DataCite/Crossref
DOI + permanent public page**; the metadata's related-identifier / references field carries an outbound
link to the brand site. Indexed by Google Scholar / OpenAIRE / Crossref → strong, durable citations:
- **Zenodo** (CERN/OpenAIRE) — `POST /api/deposit/depositions` → bucket upload → `actions/publish`.
- **OSF** — `api.osf.io/v2` project + files; **Figshare** — `/v2/account/articles`; **protocols.io**
  — publish a method (ideal for the enzyme-cascade / probiotic-paste protocol); **Harvard Dataverse**
  — `X-Dataverse-key`, dataset + Related Publication link.
- **ORCID** — a real R&D-author record; `researcher-urls` via 3-legged OAuth `/person/update`.
The one manual step is account signup (captcha) + issuing the token; everything after is scripted.

**Indexing / feed submission (no backlink, but accelerates crawl & AI-search inclusion — do these
too, they're pure `yes-now`):**
- **IndexNow** (`api.indexnow.org` / `bing.com/indexnow`, covers Bing · Yandex · Seznam · Naver) —
  host a `<key>.txt` at site root, then POST `{host,key,keyLocation,urlList}`. Submit **all locale
  URLs** so non-English markets get crawled (Yandex → RU/UK/TR, Seznam → CZ). *Proven.*
- **Google Indexing API** — `yes-with-account` (service-account JSON); officially job-posting/
  livestream only, use with care.
- **WebSub/PubSubHubbub** + **Ping-O-Matic** — `yes-now` but require the site to expose an RSS/Atom
  feed; feed-update pings, weak/transient public footprint.

**Rejected:** DBpedia (read-only downstream of Wikipedia), EAN-Search (no open write), Gravatar /
about.me (captcha), OpenStreetMap (physical-place mapping — not a citation channel), Mastodon
(captcha on open instances). Never use link-farm / auto-directory-blast tools.

## Output

Always provide: a table of channels found (with automatable rating + method), the list of links
actually created (platform, live URL, link target), any credentials to store, verification status per
link, and the `no` channels handed off with paste-ready copy.

## Common Mistakes

- Submitting to captcha/verify-gated sites via bot (fails or gets flagged) — classify first.
- Miscategorizing products into the wrong open-DB sibling (junk data, gets reverted).
- Fabricating entries or spamming duplicates — penalty and account ban risk.
- Storing new credentials in chat/memory instead of the secret store.
- Counting a submission before verifying the public record + outbound link are live.
