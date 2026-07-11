# Domain Authority Tracker

## Purpose

Establish a defensible off-site authority **baseline** and run a repeatable **monthly measurement
loop** using only free tools — so DA/DR growth is proven with dated evidence, not asserted. The
measurement half of any domain-authority / link-building program.

## When to Use

Use this skill when the task is to:

- set a starting baseline before a link-building or GEO campaign;
- track referring domains, DR/DA, and ranking movement over time;
- produce a monthly authority snapshot / dashboard;
- prove (or disprove) that off-page work is moving the numbers.

## When Not to Use

Do not use when the user needs on-page technical diagnosis (use `technical-seo-auditor`) or paid-tool
deep backlink audits (this skill is deliberately free-tool, zero-cash).

## Inputs

Domain, target market/keywords, any Google Search Console access, and the prior snapshot doc if one
exists.

## Core Principle

**One metric of record, logged forever.** Pick a single tool-independent metric (referring-domain
count) as the source of truth and never delete a row — trend continuity survives tool changes and
rate limits. DR/DA are secondary corroboration.

## Track every market, not just English

Authority and rankings are per-language. Spot-check target keywords and read GSC clicks/impressions
**per locale the site serves** (mirror its actual locale set), and tag referring domains by
language/market so the snapshot shows geographic spread — not an English-only view. A market moving
from 0 to a handful of local-language citations is real progress the global DR number can hide.

## Workflow

1. **Baseline (once).** Capture with free checkers:
   - DR + referring domains — Ahrefs free Website Authority Checker.
   - DA + linking root domains + spam score — Moz free Domain Analysis.
   - Indexed pages / sitemap URL count; GSC index coverage if available.
   Record all in a dated baseline doc; set the target (e.g. baseline + 40 referring domains).
2. **Fix the tracker.** A single append-only table: date, DR, DA, referring domains (per tool),
   Δ referring domains, ranking spot-checks, GSC clicks/impressions, notes.
3. **Monthly snapshot.** Re-pull the same free checkers, spot-check target-keyword positions, read
   GSC clicks for the priority cluster, append one dated row, compute the delta.
4. **Interpret.** Flag spam-score / manual-action warnings; note which link-building tactics moved
   referring domains; feed winners back into the acquisition plan.
5. **Acceptance.** Track against explicit criteria (N dated snapshots, ≥X net-new referring domains,
   DR/DA above baseline, target keyword in top-10, zero toxic-link warnings).

## Output

A dated baseline block + an append-only monthly snapshot table, each row with the delta and a
one-line read of what changed. Never overwrite prior rows.

## Common Mistakes

- Comparing numbers across different tools' indexes (Ahrefs vs Moz counts differ — log both, compare
  each to itself).
- Skipping the baseline, so later uplift can't be proven.
- Treating a captcha/rate-limit failure as data loss — fall back to an alternate free checker and
  keep the referring-domain count continuous.
- Reporting DA movement without checking spam score / manual actions.
