# Channel Strategies

## Goal

Choose the most reliable collection route for each platform while minimizing brittle scraping.

## Decision Table

| Platform | First Choice | Escalation | Notes |
| --- | --- | --- | --- |
| YouTube | Static page parsing of channel and watch pages | Browser only if channel listing fails | Usually enough for title, views, publish date, and description |
| Bilibili | Browser-visible uploader video page | Page scroll or second page only if needed | Public APIs are often rate-limited or risk-controlled |
| TikTok | Browser path via web-access when static page has empty `itemList` | Public mirrors only as fallback | Mirrors can help with metrics but are secondary evidence |

## YouTube Playbook

1. Open `/videos` or `/shorts`.
2. Extract visible video list.
3. Open each candidate watch page.
4. Read `publishDate`, `title`, `viewCount`, and description.
5. Keep rows only inside the requested date window.

## Bilibili Playbook

1. Open `/video` or `/upload/video`.
2. Read the first page’s video cards.
3. If the first page already spans the requested dates, stop.
4. Capture `BV` link, visible views, visible comment count, and card title.
5. If a single link is missing but the card exists, use the uploader page link and note it.

## TikTok Playbook

1. Try static profile fetch.
2. If `itemList` is empty, switch to web-access browser simulation.
3. Inspect profile page for rendered cards and visible metrics.
4. If the browser page still fails, search for public mirrors or analytics pages.
5. Mark mirror-derived values as approximate.

## Classification Rules

- `无更新`: a trustworthy source shows no posts inside the window.
- `未检出`: search did not find evidence, but proof is weak.
- `未稳定拉全`: some rows are visible but completeness is uncertain.
- `镜像估算`: metrics came from a non-canonical mirror.
