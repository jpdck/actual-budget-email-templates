# Actual Budget - Email Templates

HTML email body templates for the automated Actual Budget workflows. Each
file is a standalone HTML fragment (no `<html>`/`<head>`) meant to be used
as the `body` for `thunderbird-mail:sendMail` with `isHtml:true`.

All styling is inline (email-client-safe, no external CSS/JS). Placeholders
are `{{UPPER_SNAKE_CASE}}` - replace them with real values before sending.
Row templates for repeating table rows are given as HTML comments directly
beneath each `{{..._ROWS}}` / `{{..._LIST}}` placeholder. Every repeating
placeholder now takes `<tr>` rows (no more `<li>` lists).

## Design

Built on the **jpdck** design system's AutomationEmail layout: signal rule,
`HOST / JOB` mono eyebrow with a status tag (OK / WARN / FAIL / INFO), a
one-line summary that answers "do I need to do anything?", a mono key-value
readout, the sections, and a footer naming the sending machine. Single
column, 600px max, table-based, literal hex, light values only. Fonts are
Archivo / Martian Mono with Helvetica Neue, Arial, Menlo and Consolas as the
real email faces. FAIL variants put the readout and detail on the soft red.

- **Subjects:** `[ribbonOS] <job>: <STATUS> - <YYYY-MM-DD>`.
- **Dates:** `{{DATE}}`, `{{START_DATE}}`, `{{END_DATE}}` are ISO 8601
  (`2026-10-12`); `{{TIME}}` is 24h (`15:00`).
- **Copy:** short, no em dashes, no emoji. The summary line is the headline.
- **Order:** summary, then the run readout, then sections in the priority
  order below.

There are four independent template families, one per workflow:

## Daily Refresh (categorization)

| Situation | File | Subject |
|---|---|---|
| Step 1 failed (can't reach Actual Budget / open budget file) | `templates/failed.html` | `[ribbonOS] daily refresh: FAIL - <date>` |
| Sync failed/timed out, review still ran | `templates/sync_issue.html` | `[ribbonOS] daily refresh: WARN - <date>` |
| Ran clean, nothing to review | `templates/all_clear.html` | `[ribbonOS] daily refresh: OK - <date>` |
| Ran clean, N items flagged | `templates/needs_review.html` | `[ribbonOS] daily refresh: WARN - <date>` |

### Section rules (Daily Refresh)

- **Status banner**: always present, always first.
- **Needs your attention**: only if 1+ flagged transactions. When present,
  it goes near the top (above Run details) since it's the most actionable
  section. Omit the whole block - heading and table - if there's nothing to flag.
- **Run details**: always present, except on the hard-FAILED variant (skipped
  entirely there since no data was read).
- **Auto-categorized**: only if 1+ transactions were categorized this run.
- **New rules created**: only if 1+ rules were created this run.
- **Failure detail**: only on FAILED / Sync Issue variants.

### Files (Daily Refresh)

- `templates/failed.html` - hard failure variant
- `templates/sync_issue.html` - sync issue variant (all optional sections included, for reference)
- `templates/all_clear.html` - clean run, nothing to review
- `templates/needs_review.html` - clean run, N items flagged

## Weekly Budget Check (Wins / Cautions / Bending-to-Breaking)

| Situation | File | Subject |
|---|---|---|
| Step 1 failed (can't reach the budget tools / mcp-email-server) | `templates/weekly_failed.html` | `[ribbonOS] weekly check: FAIL - <date>` |
| Ran clean, nothing to flag | `templates/weekly_all_clear.html` | `[ribbonOS] weekly check: OK - <date>` |
| Ran clean, N Cautions/Bending-to-Breaking items | `templates/weekly_needs_attention.html` | `[ribbonOS] weekly check: WARN - <date>` |

The reader-facing heading for "Cautions" is **Worth a heads-up**, and for
"Budget Bending-to-Breaking" it's **Getting tight** - friendlier phrasing for
the same tiers used in the agent's own analysis. Tone throughout is direct
and honest, not alarmist: state the fact, then (for flagged items) a short
supportive nudge - never a scold.

### Section rules (Weekly Budget Check)

- **Status banner**: always present, always first.
- **Run details**: a single condensed comparison-summary line, present on
  every variant except FAILED.
- **Getting tight** (Bending-to-Breaking): only if 1+ items. The most urgent
  section - placed first when present.
- **Worth a heads-up** (Cautions): only if 1+ items. Placed after Getting tight.
- **Wins**: only if 1+ genuine, notable wins. Placed last on the Needs
  Attention variant (urgency first); the only data section on All Clear.
- **Failure detail**: only on the FAILED variant.

Never render an empty table or a "None" placeholder - omit the section
(heading + table) entirely instead.

### Files (Weekly Budget Check)

- `templates/weekly_failed.html` - hard failure variant
- `templates/weekly_all_clear.html` - clean run, nothing flagged (Wins optional)
- `templates/weekly_needs_attention.html` - clean run, N items flagged (all three sections shown, for reference)

## Monthly Budget Cleanup (uncategorized + duplicate sweep)

| Situation | File | Subject |
|---|---|---|
| Setup failed (tools, budget file, or MEMORY.json) | `templates/monthly_failed.html` | `[ribbonOS] monthly cleanup: FAIL - <date>` |
| Ran clean, nothing to flag | `templates/monthly_all_clear.html` | `[ribbonOS] monthly cleanup: OK - <date>` |
| Ran clean, N items flagged for review | `templates/monthly_needs_review.html` | `[ribbonOS] monthly cleanup: WARN - <date>` |

`monthly_failed.html` covers a setup failure, with an optional
`{{CONSECUTIVE_NOTE}}` line once it has failed 3+ months running. "Auto-fixed" covers both auto-categorized transactions and
auto-reconciled duplicate pairs (the agent only auto-fixes when it's fully
confident); "Needs your eyes" covers anything ambiguous - uncertain
category, possible-but-not-certain duplicate, unusual amount - left
untouched for Jeff to review.

### Section rules (Monthly Budget Cleanup)

- **Status banner**: always present, always first, holds the one-line summary.
- **Needs your eyes**: only if 1+ flagged items. The most actionable section
  - placed near the top when present.
- **Run details**: always present (window swept, sync status, transactions reviewed).
- **Auto-fixed**: only if 1+ changes were made.

Never render an empty table/list or a "None" placeholder - omit the section
(heading + table/list) entirely instead. If both Auto-fixed and Needs your
eyes are empty, the summary line should say so plainly and the email stays
to a couple of sentences.

### Files (Monthly Budget Cleanup)

- `templates/monthly_failed.html` - setup failure
- `templates/monthly_all_clear.html` - clean sweep, nothing to flag (Auto-fixed optional)
- `templates/monthly_needs_review.html` - clean sweep, N items flagged for review

## Hourly Receipt Processor (receipt mailbox)

| Situation | File | Subject |
|---|---|---|
| Step 2 failed (can't reach the mailbox / Actual Budget) | `templates/hourly_failed.html` | `[ribbonOS] receipts: FAIL - <date> <time>` |
| Run that logged something | `templates/hourly_summary.html` | `[ribbonOS] receipts: OK - <date> <time>` |

Unlike the other three workflows, there's only one non-failure variant here:
the summary template covers everything from an empty inbox to several
transactions logged and a couple of emails flagged, since this runs hourly
and a full outcome/no-outcome split would be more noise than signal at that
cadence. Skipped (non-receipt) emails are always a one-line count, never an
itemized list - newsletters and personal mail aren't worth enumerating.

### Section rules (Hourly Receipt Processor)

- **Status banner**: always present, always first, holds the one-line summary.
- **Logged**: only if 1+ transactions were created this run.
- **Needs a reply**: only if 1+ emails needed a follow-up question.
- **Failure detail**: only on the FAILED variant.

Never render an empty list or a "None" placeholder - omit the section
(heading + list) entirely instead.

### Files (Hourly Receipt Processor)

- `templates/hourly_failed.html` - hard failure variant
- `templates/hourly_summary.html` - normal run, any outcome (Logged / Needs a reply optional)

## Monthly Review (read-only month in review)

| Situation | File | Subject |
|---|---|---|
| Monthly review report | `templates/monthly_budget_review.html` | `[ribbonOS] monthly review: INFO - <date>` |

A full HTML document (it carries a `<head>` for the phone stacking rule) with
`{{ snake_case }}` placeholders: big spent number with budget bar, income /
saved / save rate, five category bars plus a ledger, biggest movers, the one
move this month, a three-step plan, and a link to the full report. The
over-budget styling (amber bar + OVER tag) is shown on `cat_3`; apply it to
whichever category is actually over. `{{ category_summary }}` replaces the
old hardcoded "four categories held the line" line.
