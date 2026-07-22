---
name: emailer
description: Das Experten universal email delivery gate — the ONLY outbound path, via Resend API (apex dasexperten.com human mail, my.dasexperten.com system mail; inbound archives to R2). ALWAYS trigger when user asks to send/draft email ("отправь письмо", "напиши и отправь", "шли по почте", "send email", "email this to", "отправь презентацию/инвойс/контракт клиенту"), OR via inter-skill gate [[GATE: emailer]] from any skill that needs outbound mail (personizer, invoicer, logist, legalizer, das-presenter, ugc-master, review-master, seo-master, XA/outreach). **sales-hunter is NOT linked and is NEVER auto-invoked for email.** Handles attachments (base64 or R2 links), virtual-staff persona resolution, mandatory pre-send confirmation gate (full email shown, waits for explicit "ок"/"шли"/"send"). Fire immediately on trigger.
---

# EMAILER v4.0 — Cloudflare + Resend only

> **v4.0 (2026-07-11), owner decision:** Google Workspace, Apps Script bridge и
> Gmail-пайплайн ВЫВЕДЕНЫ из эксплуатации. Единственный стек: Cloudflare
> (приём + архив R2) и Resend (отправка). Прежний SKILL.md v3.x — архив,
> его Apps Script/GCP/OAuth-секции больше не действуют.

Das Experten universal email delivery gate. Fire immediately when the user asks to send/draft email, or via `[[GATE: emailer]]` from any skill that needs mail (personizer, invoicer, logist, legalizer, das-presenter, ugc-master, review-master, XA/outreach, etc.). **sales-hunter is separate — only if Owner names it.**

**Direct triggers:** «отправь письмо», «напиши и отправь», «шли по почте», "send email", "email this to", «отправь презентацию/инвойс/контракт/PL клиенту», any outbound email/draft/reply request.

**Gate triggers:** `[[GATE: emailer]]` after invoicer (PDF), das-presenter (PPTX), legalizer (contract/annex), personizer (follow-up), logist (shipping docs), XA/GEO outreach drafts. **Not** after sales-hunter unless Owner explicitly ordered a sales-hunter task *and* a separate emailer send.

---

## LAZY-LOAD ROUTER

### references/virtual-staff.md (router) + per-inbox sub-references
Load router when: "sign as", «пиши от имени», «от кого письмо», persona not pre-named, «кто подписывает». Router resolves inbox → load only the matching sub-reference from `references/virtual-staff/` (eurasia, emea, sales, support, asean, marketing, partnerships, hello, agents). Do NOT preload; do NOT load for plain sends.

### references/inbox-cleanup.md — ⚠️ LEGACY
Applies ONLY to the personal Gmail `dasexperten@gmail.com` (Gmail MCP). Corporate mail is NOT in Gmail anymore — cleanup patterns there не применимы. Load only for personal-Gmail cleanup requests.

---

## 0.0. EXCLUSIVITY LOCK — non-negotiable

This skill is THE ONLY path for any outbound email from any Das Experten operation.
**sales-hunter (Owner rule):** **Отдельный скилл поиска клиентов** — not a mailer, not a presenter. Run only when Owner names sales-hunter. It may hand off to **das-presenter** (deck), **emailer** (send via virtual-staff / ALLOWED_FROM), **personizer**, etc. **Send always = this emailer skill only.**

**sales-hunter (Owner rule):** NEVER execute sales-hunter unless the user/Owner explicitly asks for sales-hunter by name. sales-hunter is NOT a mailer and is NOT part of the emailer path. Outbound mail = this skill only.

**FORBIDDEN — never use for ANY corporate email operation:**
❌ Apps Script Web App (`script.google.com/macros/.../exec`) — DECOMMISSIONED 2026-07-11.
❌ `emailer-bridge` Cloudflare Worker (Apps Script proxy) — DECOMMISSIONED, do not call.
❌ Gmail MCP tools (`Gmail:create_draft`, send, reply) for ANY @dasexperten.com / @my.dasexperten.com mail. Gmail MCP остаётся только для личного ящика dasexperten@gmail.com / a.v.badalyan@gmail.com по прямой просьбе Арама.
❌ `Make:*` email agents.
❌ Any other ESP (SendGrid, Mailgun, SES direct, Postmark…) — unmanaged sender identities.

**ALLOWED — the only path:**
✅ Resend API (`api.resend.com/emails`) with the send-only key, from-адрес из Section 2.2. Attachments: inline base64 или R2-ссылки (Section 5).

**Translation rule:** any sub-skill spec saying "save Gmail draft", "Apps Script", "emailer-bridge" — читать как `[[GATE: emailer]]` по этому файлу. Conflicts → this lock wins; flag the outdated sub-skill to Aram.

**[TODO — flips automatically after Release 2 of plan-emailer-dark-ui]:** primary path becomes `POST dasoperator-api /api/email/send` (Resend + R2-архив + D1-лог серверно). Until then sends go direct-to-Resend and are NOT auto-archived to R2 — always fill `context` and report message id in chat (audit trail = чат + Resend dashboard + D1 emails_sent, где применимо).

---

## 0. ABSOLUTE SAFETY GATE — non-negotiable (unchanged)

Before EVERY actual send:
1. Display full block: recipient(s), from-persona, subject, full body (no truncation), attachments with sizes, `context`.
2. Line: `Подтверди отправку: "ок" / "шли" / "отправляй"`
3. **STOP.** Wait for the next user message.
4. Proceed only on: `ок`, `шли`, `отправляй`, `send`, `да`, `+`, `go`, `confirm`, `подтверждаю`, `можно`.
5. Edits → apply, redisplay, wait again. Ambiguous → redisplay, ask again.
6. Legal documents (legalizer) → prefix `⚠️ LEGAL DOCUMENT DELIVERY — irreversible action`.

**Hard rule:** no automatic sends, no fire-and-forget. Черновики (draft_only) в v4 = письмо показано в чате и НЕ отправлено (Resend черновиков не имеет); хранение черновика = сам чат/файл.

---

## 1. INPUT PARAMETERS

**Required:** `recipient`; `subject`; `body_plain` и/или `body_html`.
**Optional:** `attachments_local_paths` (sandbox paths → base64 inline или R2), `attachments_urls` (public URLs → fetch → base64 inline; крупные — ссылкой в теле), `from` (см. 2.2; default sales@dasexperten.com), `from_name` (display name персоны, см. virtual-staff), `cc`, `bcc`, `reply_to`, `context` (обяз. для аудита при gate-вызовах), `draft_only`, `legal`, `in_reply_to` + `references` (Message-ID цепочки для тредов — брать из R2-архива входящего письма).

**Reply mode:** Gmail `thread_id` больше НЕ существует. Ответ = обычный send с `subject: Re: …`, `in_reply_to`/`references` заголовками и `from` = адрес, на который пришло письмо (смотри запись в R2 Inbox/<address>/received/).

---

## 2. PIPELINE

```
Calling skill → validate → resolve attachments → confirmation gate → WAIT "ок"
→ POST api.resend.com/emails → parse → report (id + persona + attachments)
```

### 2.1. Credentials — self-contained

| Component | Value |
|---|---|
| Resend API endpoint | `https://api.resend.com/emails` |
| **RESEND_API_KEY (send-only)** | → `SECRETS/resend.md` (никогда не хардкодить здесь; читать перед send) |
| Полный реестр Resend (домены, ID, план Pro) | `SECRETS/resend.md` |
| R2 bucket (крупные вложения, ссылки) | `emailer-attachments`, public `https://pub-0e2fb2d28ea9408bbaa1bdd64b3bf256.r2.dev/` |
| R2 API token | → `SECRETS/cloudflare.md` (scope R2 Edit; никогда не хардкодить здесь) |
| CF Account ID | `081ddb85cb399ad62a70210328d744fc` |
| Входящие (архив, чтение) | R2 bucket `self-learning` → `Inbox/<address>/…` (binding ARCHIVE в dasoperator-api) |
| План Pro | 50 000 писем/мес, дневного лимита нет; overage $0.90/1000 |

Auth: `Authorization: Bearer re_…`. На 401/403 — стоп, назвать упавший credential по имени, ждать ротации.

### 2.2. ALLOWED_FROM — sender registry (2026-07-11)

**Человеческие (апекс dasexperten.com, Resend-verified, DKIM d=dasexperten.com — без «via»):**

| Address | Purpose | Virtual staff |
|---|---|---|
| `sales@dasexperten.com` | B2B-продажи, дистрибьюторы, опт | `virtual-staff/sales.md` |
| `support@dasexperten.com` | Постпродажный сервис, возвраты, ретеншн | `virtual-staff/support.md` |
| `emea@dasexperten.com` | Не-русскоязычный хаб: EN/DE/IT/ES/AR | `virtual-staff/emea.md` |
| `eurasia@dasexperten.com` | Русскоязычный хаб: СНГ | `virtual-staff/eurasia.md` |
| `asean@dasexperten.com` | Юго-Восточная Азия: VN/TH/MY/PH/ID/SG | `virtual-staff/asean.md` |
| `dr.badalyan@dasexperten.com` | Личный адрес Арама — подписывается ТОЛЬКО сам Арам, без виртуальных персон; использовать только по его прямому указанию | — |
| `marketing@dasexperten.com` | Блогеры/UGC-коллаборации, ретейл-медиа, платформы/CRM/маркетплейс-менеджеры, бренд-контент | `virtual-staff/marketing.md` |
| `partnerships@dasexperten.com` | SEO/GEO-аутрич: линкбилдинг, гостевые публикации, PR-питчи, каталоги, affiliate, контент-партнёрства | `virtual-staff/partnerships.md` |
| `hello@dasexperten.com` | Тёплый фронт бренда: регистрации в каталогах/агрегаторах, переписка с владельцами сайтов, первое касание вне sales-контекста | `virtual-staff/hello.md` |

**Именные (Founding Ten department heads, апекс dasexperten.com — 5 из 10, Owner decision 2026-07-13):**

| Address | Purpose | Virtual staff |
|---|---|---|
| `lauda@dasexperten.com` | Head of Commerce — сделки/дистрибьюторы, которыми владеет Lauda Briana | `virtual-staff/agents.md` |
| `roberta@dasexperten.com` | Head of Content — SEO/гостевые публикации/контент-партнёрства уровня head | `virtual-staff/agents.md` |
| `marika@dasexperten.com` | Head of Brand — креативные/брендовые коллаборации | `virtual-staff/agents.md` |
| `valentina@dasexperten.com` | Head of Legal — контракты/NDA (legalizer) | `virtual-staff/agents.md` |
| `justina@dasexperten.com` | Head of Finance — инвойсы/юнит-экономика (invoicer) | `virtual-staff/agents.md` |

Именной ящик = фиксированное реальное имя, без gender-mirror замены. Остальные пять Founding Ten (`lena@`, `alexandra@`/`sasha@`, `mina@`, `zina@`, `maya@`) — **reserved, ящик не провижен**, не использовать как `from`.

**Системные (my.dasexperten.com, Resend-verified):** `orders@`, `forms@`, `no-reply@`, `notifications@`, `system@` — шлют ТОЛЬКО Workers по триггерам. Руками/skill'ом с них не отправлять.

`marketing@`, `partnerships@`, `hello@` — добавлены 2026-07-13 решением Арама. ⚠️ До завершения инфраструктуры (CF Email Routing маршруты + Resend from-верификация, чеклист в Section 8 changelog) отправка с них ЗАПРЕЩЕНА — при запросе сообщить статус и предложить временный fallback sales@ (только с явного одобрения Арама).
`lauda@`…`justina@` — апекс-домен уже verified в Resend (отдельная верификация адреса не нужна для отправки), но CF Email Routing (приём входящих на эти 5 адресов) не подтверждён в SECRETS/cloudflare.md — проверить перед тем, как обещать получателю ответ на этот адрес.

Любой `from` вне реестра → отказ до отправки, спросить Арама.

---

## 3. STEP-BY-STEP EXECUTION

### Step 1 — Validate
recipient regex `^[^\s@]+@[^\s@]+\.[^\s@]+$`; subject non-empty; body non-empty; from ∈ реестра 2.2. Fail → вернуть ошибку вызвавшему скиллу, Арама не дёргать.

### Step 2 — Attachments
- Файл ≤ 10 MB: читать → base64 → элемент `attachments: [{filename, content}]` (Resend total limit 40 MB после кодирования; держать суммарно ≤ 25 MB).
- Файл крупнее / много файлов: PUT в R2 `emailer-attachments` (паттерн имени `{skill}_{YYYY-MM-DD}_{6hex}.{ext}`, Content-Type по таблице расширений) → публичная ссылка строкой в теле письма, пометить в confirmation-блоке «уйдёт ссылкой».
- Ошибка загрузки → сообщить, abort. Молча не пропускать.

### Step 3 — Payload
```json
{
  "from": "James Carter — Das Experten <sales@dasexperten.com>",
  "to": ["client@example.com"],
  "cc": [], "bcc": [],
  "reply_to": "sales@dasexperten.com",
  "subject": "Das Experten — distributor proposal",
  "text": "…", "html": "…",
  "headers": { "In-Reply-To": "<msgid>", "References": "<msgid>" },
  "attachments": [ { "filename": "proposal.pdf", "content": "<base64>" } ]
}
```
`from` display name = выбранная персона (virtual-staff) или `Das Experten`.

### Step 4 — Confirmation block (см. SAFETY GATE). Step 5 — STOP, ждать.

### Step 6 — Send
```bash
RESEND_API_KEY=$(grep -o 're_[A-Za-z0-9_]*' SECRETS/resend.md | head -1)  # из SECRETS, не из этого файла
curl -sS -X POST https://api.resend.com/emails \
  -H "Authorization: Bearer $RESEND_API_KEY" \
  -H "Content-Type: application/json" \
  --max-time 60 -d @payload.json
```

### Step 7 — Parse & report
Success: `{"id":"<uuid>"}` →
```
✅ Отправлено от <persona> <from>
   resend id: <uuid>
   вложения: N (имена) / ссылкой
```
Errors: 422 validation (поле в `message`) — исправить, повторить цикл с подтверждением; 401/403 — ключ, стоп; 429 — подождать 30 с, одна повторная попытка; `domain is not verified` — проверить `SECRETS/resend.md`, стоп. Никаких автоповторов сверх одного 429-ретрая.

---

## 4. INTER-SKILL CONTRACT (unchanged)
Calling skill даёт параметры Section 1 (можно локальные пути — emailer сам решает base64/R2). НЕ зовёт curl сам, НЕ обходит confirmation gate. Возврат: `resend_id` или ошибка; правки Арама — внутренний цикл emailer.

## 5. SKILL-SPECIFIC PATTERNS (unchanged intents, новый транспорт)
invoicer → PDF-вложения, from sales@ или eurasia@ по языку; das-presenter → PPTX (обычно ссылкой, >10 MB), from sales@/emea@/asean@ по региону; legalizer → `legal: true`; sales-hunter → **own client-hunt skill**; handoff → **das-presenter** (deck) / **emailer** (send + virtual-staff) / personizer → reply-цепочка через in_reply_to из R2-архива; logist → документы отгрузки; seo-master → линк-аутрич/PR-питчи, from partnerships@; ugc-master → блогерский бартер, from marketing@ (UGC override: женская персона); регистрации в каталогах/сайтовые запросы → from hello@.

## 6. INBOUND (справочно)
Приём всех адресов: MX → CF Email Routing → worker dasoperator-api → R2 `self-learning` Inbox/<address>/received/. Чтение переписки — UI /emailer в Das Operator или чтение R2. Этот skill входящие не обрабатывает — только отправка.

## 7. TROUBLESHOOTING
- Письмо «via amazonses»/в спаме → проверить DNS: `resend._domainkey.dasexperten.com` TXT, `send.dasexperten.com` TXT/MX (см. SECRETS/resend.md), статус домена в Resend.
- 429 / лимит → план Pro 50k/мес; смотреть usage в Resend dashboard.
- R2 401 → токен R2 из 2.1; fallback CF Cloud Master (SECRETS/cloudflare.md).
- Ответ не склеился в тред у получателя → заполнены ли In-Reply-To/References из исходного письма (R2-запись, поле messageId).

## 8. CHANGELOG
- **v4.2 (2026-07-16)** — ALLOWED_FROM +5 именных Founding Ten mailboxes: `lauda@` (Commerce), `roberta@` (Content), `marika@` (Brand), `valentina@` (Legal), `justina@` (Finance) — источник `dasexperten/organizacia` (Personnel SSOT). Новый sub-reference `virtual-staff/agents.md` (+ `agent-avatars.md` для avatar hard rule). Ранее эти файлы существовали только в копии скилла внутри `organizacia` и не были в реестре ALLOWED_FROM ни в одной рабочей копии — синхронизировано по всем зеркалам скилла.
- **v4.1 (2026-07-13)** — SECURITY: живые Resend/R2 credentials вынесены из SKILL.md в SECRETS/ (GitHub push protection их блокировал — подтверждение аудита);  ALLOWED_FROM +3: marketing@ (восстановлен), partnerships@ (SEO/GEO-аутрич), hello@ (тёплый фронт). Новые sub-references: partnerships.md, hello.md; marketing.md реактивирован на собственный ящик. Инфра-чеклист до первой отправки: (1) CF Email Routing — маршруты приёма 3 адресов → dasoperator-api; (2) Resend — from-адреса на верифицированном апексе (домен уже verified, отдельная верификация адресов не нужна — проверить sending готовность); (3) снять ⚠️-блок в Section 2.2. До снятия блока отправка запрещена.
- **v4.0 (2026-07-11)** — Cloudflare + Resend only. Apps Script/GCP/OAuth/emailer-bridge удалены. ALLOWED_FROM → 6 адресов апекса .com + системные my. Черновики = показ в чате. Reply через In-Reply-To. Virtual staff: +asean, dr.badalyan (только Арам), marketing@ декомиссован.
- v3.2 (2026-05-11) — архивная версия (Apps Script), см. git-историю.
