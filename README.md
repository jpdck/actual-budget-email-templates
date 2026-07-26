# Actual Budget Daily Refresh — Email Templates

HTML email body templates for the daily Actual Budget categorization workflow.
Each file is a standalone HTML fragment (no `<html>`/`<head>`) meant to be
used as the `body` for `thunderbird-mail:sendMail` with `isHtml:true`.

All styling is inline (email-client-safe, no external CSS/JS). Placeholders
are `{{UPPER_SNAKE_CASE}}` — replace them with real values before sending.
Row/list templates for repeating table rows are given as HTML comments
directly beneath each `{{..._ROWS}}` / `{{..._LIST}}` placeholder.

## Picking a variant

| Situation | File | Subject |
|---|---|---|
| Step 1 failed (can't reach Actual Budget / open budget file) | `templates/failed.html` | `Actual Budget -- Daily Refresh: FAILED (<date>)` |
| Sync failed/timed out, review still ran | `templates/sync_issue.html` | `Actual Budget -- Daily Refresh: Sync Issue (<date>)` |
| Ran clean, nothing to review | `templates/all_clear.html` | `Actual Budget -- Daily Refresh: All Clear (<date>)` |
| Ran clean, N items flagged | `templates/needs_review.html` | `Actual Budget -- Daily Refresh: N Need Review (<date>)` |

## Section rules (apply across all variants)

- **Status banner**: always present, always first.
- **Needs your attention**: only if 1+ flagged transactions. When present,
  it goes near the top (above Run details) since it's the most actionable
  section. Omit the whole block — heading and table — if there's nothing to flag.
- **Run details**: always present, except on the hard-FAILED variant (skipped
  entirely there since no data was read).
- **Auto-categorized**: only if 1+ transactions were categorized this run.
- **New rules created**: only if 1+ rules were created this run.
- **Failure detail**: only on FAILED / Sync Issue variants.

Never render an empty table or a "None" placeholder — omit the section
(heading + table/list) entirely instead.

## Files

- `templates/failed.html` — hard failure variant
- `templates/sync_issue.html` — sync issue variant (all optional sections included, for reference)
- `templates/all_clear.html` — clean run, nothing to review
- `templates/needs_review.html` — clean run, N items flagged
