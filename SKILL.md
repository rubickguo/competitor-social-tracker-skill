---
name: competitor-social-tracker
description: Collect weekly or custom-range competitor social-media posts across YouTube, TikTok, Bilibili, and similar public video channels, then normalize them into a report table and Excel file. Use when the user provides competitor names, channel URLs, date ranges, or wants a recurring competitor tracking report with fields such as channel, video link, title, translated title, item or cosmetic name, summary, views, likes, comments, and comment direction. Especially use this skill when data acquisition method matters by channel, such as static extraction for YouTube and browser-simulated access via the web-access skill for TikTok.
---

# Competitor Social Tracker

## Overview

Build a competitor content tracker from public social platforms and export it as a table plus `.xlsx`.
Prefer the lightest reliable collection path for each platform, and escalate to browser-simulated access only when static fetches are incomplete.

## Inputs

Expect or infer these inputs:

- competitor label, such as `FF`, `PUBGM`, `Genshin`
- channel type, such as `YouTube`, `TikTok`, `Bilibili`
- channel URL or account handle
- date window with absolute dates
- output columns; default to:
  `channel | account_name | publish_date | source_title | translated_title_zh | video_link | item_or_topic | summary | views | likes_comments | comment_direction | source_note`

If the user asks for "Excel", always also create a machine-friendly TSV intermediate and then export `.xlsx`.

## Workflow

1. Normalize the request.
Use absolute dates.
Expand each source into `platform + URL + account label + date window`.

2. Choose the acquisition route per platform.
See [references/channel-strategies.md](references/channel-strategies.md).

3. Collect the weekly post list first.
Do not summarize before you have the full list for that account in the date window.
For each row, capture at minimum:
`published date`, `canonical link`, `title`, `visible metrics`, and `short description or visible summary`.

4. Enrich the rows.
Translate title to Chinese when needed.
If the user wants item-level tracking such as skins or ornaments, extract the item name from title, description, or post text and place it in `item_or_topic`.
If the row is not item-specific, use a concise topic label instead.

5. Add quality notes.
If metrics come from a mirror or browser-visible card rather than a canonical API response, record that in `source_note`.
If a channel cannot be verified reliably, do not invent rows. Mark it as `not_stably_complete`.

6. Export deliverables.
Write:
- markdown summary if useful
- `.tsv` for easy inspection
- `.xlsx` via `scripts/export_competitor_xlsx.py`

## Platform Rules

### YouTube

- Prefer static fetch plus structured extraction.
- Good first paths:
  - channel `/videos` or `/shorts`
  - watch page metadata
  - visible title, view count, publish date, description
- Use raw HTTP or page-source parsing first.
- Escalate to browser only if the channel page fails to reveal the needed list.
- For comments, summarize direction only when comments are stably visible. Otherwise infer direction from visible engagement and content framing, and say that it is directional.

### Bilibili

- Prefer browser-visible upload pages or public page HTML over unstable API endpoints.
- Read the uploader video page and extract the latest cards.
- If the first page covers the requested date range, do not paginate further.
- Use the canonical `BV` video link when available.
- If only the upload page card is stable, use the upload page as the link and mention that the single-video link was not stably recoverable.

### TikTok

- Treat TikTok as dynamic by default.
- Use the web-access skill at `C:/Users/admin/.codex/skills/web-access/SKILL.md` whenever static fetch does not expose `itemList` or returns empty profile data.
- Prefer this order:
  1. public profile page hydration data
  2. browser-simulated page inspection via web-access/CDP
  3. public mirrors or analytics pages only as fallback
- When using mirrors, clearly label them as mirror-derived metrics.
- If the real TikTok page and mirrors disagree, prefer real TikTok existence and mirror metrics only as approximate performance.
- If you still cannot verify weekly posts, output `not_stably_complete` instead of guessing.

## Row Construction Rules

- One post per row.
- Keep titles verbatim from source, then add a Chinese translation in the same field or adjacent field according to the user's requested schema.
- Keep content summaries short and factual.
- Do not overclaim comment sentiment; use phrases like `mostly anticipation`, `benefit-driven`, `story-discussion`, or `meme-heavy`.
- Preserve exact URLs.
- Preserve exact visible metrics with units.

## Output Schema

Default sheet name: `tracker`

Default columns:

```text
channel
account_name
publish_date
source_title
translated_title_zh
video_link
item_or_topic
summary
views
likes_comments
comment_direction
source_note
```

If the user wants fewer columns, still build the full internal table first, then drop columns at export time.

## Export

Use `scripts/export_competitor_xlsx.py` to convert a TSV file into `.xlsx`.

Example:

```powershell
python "C:\Users\admin\.codex\skills\competitor-social-tracker\scripts\export_competitor_xlsx.py" `
  --input "D:\share\competitor-tracker.tsv" `
  --output "D:\share\competitor-tracker.xlsx"
```

The exporter expects UTF-8 TSV with a header row.

## Failure Policy

- Prefer partial truth over fake completeness.
- If one platform is blocked, still deliver the rest and isolate the blocked channel in a `not_stably_complete` section.
- Always distinguish:
  - `no_update`
  - `not_found`
  - `not_stably_complete`
  - `mirror_estimate`

## References

- Read [references/channel-strategies.md](references/channel-strategies.md) when choosing a collection method.
- Read [references/output-template.tsv](references/output-template.tsv) when you need the canonical column order.
