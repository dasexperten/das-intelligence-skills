# Agent avatars — identity hard rule (Owner, 2026-07-13)

## HARD RULE (non-negotiable)

Wherever an agent’s **name**, **initials**, or **email signature** appears, show a **small close-up portrait avatar** of that agent.

| Requirement | Spec |
|---|---|
| Framing | **Close-up only** — face + upper chest; **maximum crop ends at the breast / décolletage line**; never full torso, never waist, never full-body. Face large in frame. |
| Expression | Prefer a warm smile when the **founding reference** already has one. **Never re-face with AI** if it breaks recognition — identity lock beats smile remakes. |
| Identity | **Hard lock:** public avatars are **crops of founding Higgsfield masters** (`organizacia` commit founding ten). No generative re-draw. Remake smile only with Owner-approved reference edit that keeps the same face. |
| Crop for UI/email | Circular or soft-rounded square, **40–48 px** in email; 32–64 px in dense UI |
| Placement | **Left of** name / initials / signature block (LTR); never floating without the name |
| Consistency | Same face asset everywhere (email, Operator UI, decks, org chart chips) |
| No photo? | Do **not** invent a new face mid-thread. Use SSOT avatar only; if missing, escalate to Marika (Brand) / Mina Rutunya (hosting) |

Applies to **all Founding Ten**, not only the five mailboxes.

---

## Asset SSOT

| Layer | Path |
|---|---|
| Master portraits | `dasexperten/organizacia` → `avatars/{slug}.png` (full-res SSOT) |
| Public CDN (email + UI) | **LIVE** — `https://www.dasexperten.com/assets/agents/{slug}.png` |
| Local deploy tree | `site/com/assets/agents/{slug}.png` (256px square) + `{slug}@2x.png` (96px) |
| Republish | `tools/_publish_agent_avatars.py` then Pages deploy `site/com` |

### Public URL examples (live 2026-07-13)

```
https://www.dasexperten.com/assets/agents/lauda-briana.png
https://www.dasexperten.com/assets/agents/roberta-di-maria.png
https://www.dasexperten.com/assets/agents/marika-nowicka.png
https://www.dasexperten.com/assets/agents/valentina-korolyeva.png
https://www.dasexperten.com/assets/agents/justina-timber.png
https://www.dasexperten.com/assets/agents/lena-sergeeva.png
```

### Slugs

| Agent | Slug | Initials |
|---|---|---|
| Lena Sergeeva | `lena-sergeeva` | LS |
| Lauda Briana | `lauda-briana` | LB |
| Alexandra Obnorskaya | `alexandra-obnorskaya` | AO |
| Roberta Di Maria | `roberta-di-maria` | RDM |
| Marika Nowicka | `marika-nowicka` | MN |
| Valentina Korolyeva | `valentina-korolyeva` | VK |
| Justina Timber | `justina-timber` | JT |
| Mina Rutunya | `mina-rutunya` | MR |
| Zina Pevtsova | `zina-pevtsova` | ZP |
| Maya Krasochkina | `maya-krasochkina` | MK |

---

## Surfaces

| Surface | How |
|---|---|
| **HTML email signature** | 40–48 px circular `<img>` left of name block (see template below) |
| **Plain-text email** | No image possible — name + role only; prefer multipart HTML when agent From is used |
| **Initials chips (UI)** | Prefer **photo avatar with initials as alt/fallback**, not letters-only |
| **PPTX / decks** | Same portrait, larger crop allowed for title slides |
| **Telegram / chat** | Same asset as profile photo when agent persona is used |

---

## HTML email signature template

Use with emailer `body_html` (multipart). Replace `{SLUG}`, `{NAME}`, `{ROLE}`, `{EMAIL}`, `{PUBLIC_AVATAR_URL}`.

```html
<table cellpadding="0" cellspacing="0" role="presentation" style="margin-top:16px;font-family:Arial,Helvetica,sans-serif;font-size:13px;color:#222;">
  <tr>
    <td style="vertical-align:top;padding-right:12px;">
      <img src="{PUBLIC_AVATAR_URL}" width="48" height="48" alt="{NAME}"
           style="border-radius:50%;display:block;border:0;outline:none;" />
    </td>
    <td style="vertical-align:middle;line-height:1.35;">
      <div style="font-weight:700;font-size:14px;color:#111;">{NAME}</div>
      <div style="color:#444;">{ROLE} | Das Experten</div>
      <div><a href="mailto:{EMAIL}" style="color:#1a5f4a;text-decoration:none;">{EMAIL}</a></div>
    </td>
  </tr>
</table>
```

Plain-text fallback (always include in multipart):

```
Best regards,
{NAME}
{ROLE} | Das Experten
{EMAIL}
```

---

## Regeneration brief (if portrait must be remade)

When generating or editing agent portraits (Higgsfield / imager / etc.):

1. **Identity lock** — match existing reference model; do not reinvent the face.
2. **Close-up** — face ≥ 50% of frame; soft natural light.
3. **Duchenne smile** — eyes engaged, cheeks lifted, not a stiff commercial grin.
4. **Background** — clean, brand-compatible (paper / soft studio); no logos behind head.
5. Output master PNG in `organizacia/avatars/{slug}.png`, then mirror to public CDN.

---

## emailer enforcement

- Agent From (`lauda@` … `justina@` and future agent boxes): **prefer HTML body** with avatar signature when sending externally.
- Until public CDN URLs are live, still attach identity rule to drafts; use plain signature only as temporary fallback and note “avatar pending public host”.
