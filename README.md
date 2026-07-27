# Actual Budget — Email Templates

HTML email body templates for the automated Actual Budget workflows. Each
file is a standalone HTML fragment (no `<html>`/`<head>`) meant to be used
as the `body` for `thunderbird-mail:sendMail` with `isHtml:true`.

All styling is inline (email-client-safe, no external CSS/JS). Placeholders
are `{{UPPER_SNAKE_CASE}}` — replace them with real values before sending.
Row/list templates for repeating table rows are given as HTML comments
directly beneath each `{{..._ROWS}}` / `{{..._LIST}}` placeholder.

There are three independent template families, one per workflow:

## Daily Refresh (categorization)

| Situation | File | Subject |
|---|---|---|
| Step 1 failed (can't reach Actual Budget / open budget file) | `templates/failed.html` | `Actual Budget -- Daily Refresh: FAILED (<date>)` |
| Sync failed/timed out, review still ran | `templates/sync_issue.html` | `Actual Budget -- Daily Refresh: Sync Issue (<date>)` |
| Ran clean, nothing to review | `templates/all_clear.html` | `Actual Budget -- Daily Refresh: All Clear (<date>)` |
| Ran clean, N items flagged | `templates/needs_review.html` | `Actual Budget -- Daily Refresh: N Need Review (<date>)` |

### Section rules (Daily Refresh)

- **Status banner**: always present, always first.
- **Needs your attention**: only if 1+ flagged transactions. When present,
  it goes near the top (above Run details) since it's the most actionable
  section. Omit the whole block — heading and table — if there's nothing to flag.
- **Run details**: always present, except on the hard-FAILED variant (skipped
  entirely there since no data was read).
- **Auto-categorized**: only if 1+ transactions were categorized this run.
- **New rules created**: only if 1+ rules were created this run.
- **Failure detail**: only on FAILED / Sync Issue variants.

### Files (Daily Refresh)

- `templates/failed.html` — hard failure variant
- `templates/sync_issue.html` — sync issue variant (all optional sections included, for reference)
- `templates/all_clear.html` — clean run, nothing to review
- `templates/needs_review.html` — clean run, N items flagged

## Weekly Budget Check (Wins / Cautions / Bending-to-Breaking)

| Situation | File | Subject |
|---|---|---|
| Step 1 failed (can't reach the budget tools / Thunderbird) | `templates/weekly_failed.html` | `Weekly Budget Check -- FAILED (<date>)` |
| Ran clean, nothing to flag | `templates/weekly_all_clear.html` | `Weekly Budget Check -- All Clear (<date>)` |
| Ran clean, N Cautions/Bending-to-Breaking items | `templates/weekly_needs_attention.html` | `Weekly Budget Check -- Needs Attention: N item(s) (<date>)` |

The reader-facing heading for "Cautions" is **Worth a heads-up**, and for
"Budget Bending-to-Breaking" it's **Getting tight** — friendlier phrasing for
the same tiers used in the agent's own analysis. Tone throughout is direct
and honest, not alarmist: state the fact, then (for flagged items) a short
supportive nudge — never a scold.

### Section rules (Weekly Budget Check)

- **Status banner**: always present, always first.
- **Run details**: a single condensed comparison-summary line, present on
  every variant except FAILED.
- **Getting tight** (Bending-to-Breaking): only if 1+ items. The most urgent
  section — placed first when present.
- **Worth a heads-up** (Cautions): only if 1+ items. Placed after Getting tight.
- **Wins**: only if 1+ genuine, notable wins. Placed last on the Needs
  Attention variant (urgency first); the only data section on All Clear.
- **Failure detail**: only on the FAILED variant.

Never render an empty table or a "None" placeholder — omit the section
(heading + table) entirely instead.

### Files (Weekly Budget Check)

- `templates/weekly_failed.html` — hard failure variant
- `templates/weekly_all_clear.html` — clean run, nothing flagged (Wins optional)
- `templates/weekly_needs_attention.html` — clean run, N items flagged (all three sections shown, for reference)

## Monthly Budget Cleanup (uncategorized + duplicate sweep)

| Situation | File | Subject |
|---|---|---|
| Ran clean, nothing to flag | `templates/monthly_all_clear.html` | `Monthly Budget Cleanup - <Month> <Year>` |
| Ran clean, N items flagged for review | `templates/monthly_needs_review.html` | `Monthly Budget Cleanup - <Month> <Year>` |

This workflow has no FAILED variant — its prompt doesn't define a hard
connectivity check the way the Daily Refresh and Weekly Budget Check
workflows do. "Auto-fixed" covers both auto-categorized transactions and
auto-reconciled duplicate pairs (the agent only auto-fixes when it's fully
confident); "Needs your eyes" covers anything ambiguous — uncertain
category, possible-but-not-certain duplicate, unusual amount — left
untouched for Jeff to review.

### Section rules (Monthly Budget Cleanup)

- **Status banner**: always present, always first, holds the one-line summary.
- **Needs your eyes**: only if 1+ flagged items. The most actionable section
  — placed near the top when present.
- **Run details**: always present (window swept, sync status, transactions reviewed).
- **Auto-fixed**: only if 1+ changes were made.

Never render an empty table/list or a "None" placeholder — omit the section
(heading + table/list) entirely instead. If both Auto-fixed and Needs your
eyes are empty, the summary line should say so plainly and the email stays
to a couple of sentences.

### Files (Monthly Budget Cleanup)

- `templates/monthly_all_clear.html` — clean sweep, nothing to flag (Auto-fixed optional)
- `templates/monthly_needs_review.html` — clean sweep, N items flagged for review
