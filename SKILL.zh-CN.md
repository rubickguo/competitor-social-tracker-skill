---
name: competitor-social-tracker
description: 用于收集 YouTube、TikTok、Bilibili 等平台在指定时间范围内的竞品社媒视频，并整理成报告表格与 Excel。适用于用户提供竞品名称、渠道 URL、时间区间，或希望持续生成竞品追踪报表的场景。适合输出字段包括渠道、视频链接、标题、中文翻译标题、饰品或主题名称、内容说明、播放量、点赞评论和评论风向。尤其适用于不同渠道需要不同抓取策略的情况，例如 YouTube 倾向静态提取，TikTok 倾向通过 web-access 技能做浏览器模拟访问。
---

# 竞品社媒追踪

## 概览

从公开社媒平台收集竞品内容，并导出为表格与 `.xlsx`。
默认优先走最轻量、最稳定的抓取路径；只有静态抓取不完整时，才升级到浏览器模拟访问。

## 输入

默认需要或应当自动补全这些输入：

- 竞品标签，例如 `FF`、`PUBGM`、`原神`
- 渠道类型，例如 `YouTube`、`TikTok`、`Bilibili`
- 渠道 URL 或账号 handle
- 使用绝对日期的时间窗口
- 输出字段；默认采用：
  `channel | account_name | publish_date | source_title | translated_title_zh | video_link | item_or_topic | summary | views | likes_comments | comment_direction | source_note`

如果用户要求“导出 Excel”，始终先生成一份机器友好的 TSV，再导出 `.xlsx`。

## 工作流

1. 规范化需求。
始终使用绝对日期。
将每个输入源展开为 `platform + URL + account label + date window`。

2. 为不同平台选择采集路线。
参考 [references/channel-strategies.md](references/channel-strategies.md)。

3. 先拿到该账号在指定时间窗内的完整视频列表。
在确认完整列表之前，不要先做摘要。
每一行至少要包含：
`发布时间`、`规范链接`、`标题`、`公开可见数据`、`简介或可见摘要`。

4. 丰富字段。
如果标题不是中文，补一个中文翻译标题。
如果用户要求追踪饰品、皮肤、套装等内容，就从标题、简介或帖子正文中提取具体物料名，写入 `item_or_topic`。
如果不是具体物料内容，则写一个简明主题标签。

5. 记录质量备注。
如果数据来自镜像站或浏览器卡片可见值，而不是规范 API 或单视频页，就写入 `source_note`。
如果某个渠道无法稳定核验，不要编造结果，标记为 `not_stably_complete`。

6. 导出交付物。
输出：
- 如有需要，附 markdown 摘要
- `.tsv` 作为检查中间产物
- 通过 `scripts/export_competitor_xlsx.py` 导出 `.xlsx`

## 平台规则

### YouTube

- 优先使用静态抓取和结构化提取。
- 默认优先路径：
  - 频道 `/videos` 或 `/shorts`
  - 单视频 watch 页面元数据
  - 页面可见标题、播放量、发布时间和简介
- 先用原始 HTTP 或页面源码提取。
- 只有当频道页拿不到完整列表时，才升级到浏览器方案。
- 评论风向只有在评论稳定可见时才做总结；否则根据公开互动表现和内容类型做方向性判断，并明确说明这是方向判断。

### Bilibili

- 优先读 UP 主投稿页可见卡片或公开 HTML，不优先依赖容易风控的 API。
- 进入 `/video` 或 `/upload/video` 页读取最新卡片。
- 如果首页已经覆盖了目标日期，就不要继续翻页。
- 能拿到 `BV` 直链时，优先使用 `BV` 链接。
- 如果单条 `BV` 链接不稳定，但卡片存在，可以退回使用投稿页链接，并在备注里说明。

### TikTok

- 默认把 TikTok 视作动态站点。
- 当静态抓取拿不到 `itemList` 或 profile 数据为空时，使用 `web-access` 技能路径：`C:/Users/admin/.codex/skills/web-access/SKILL.md`
- 默认优先级：
  1. profile 页 hydration 数据
  2. 通过 web-access/CDP 的浏览器模拟页面检查
  3. 公开镜像站或分析站作为兜底
- 使用镜像站时，必须明确标注这是镜像来源数据。
- 如果真实 TikTok 页面和镜像数据不一致，以真实 TikTok 的“是否存在该帖”优先，镜像值仅作为近似表现。
- 如果仍然无法验证周内完整帖单，输出 `not_stably_complete`，不要猜。

## 行构造规则

- 一条帖子一行。
- 标题先保留原文，再补中文翻译。
- 内容说明保持简洁、事实化。
- 不要过度宣称评论情绪，可使用类似 `偏期待`、`偏福利导向`、`偏剧情讨论`、`偏玩梗互动` 这样的表述。
- 保留精确 URL。
- 保留公开页面可见的原始数据量级和单位。

## 输出字段

默认工作表名：`tracker`

默认字段顺序：

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

如果用户要求更少字段，也先构造完整表，再在导出时裁剪列。

## 导出

使用 `scripts/export_competitor_xlsx.py` 将 TSV 转成 `.xlsx`。

示例：

```powershell
python "C:\Users\admin\.codex\skills\competitor-social-tracker\scripts\export_competitor_xlsx.py" `
  --input "D:\share\competitor-tracker.tsv" `
  --output "D:\share\competitor-tracker.xlsx"
```

导出脚本要求输入为 UTF-8 编码、带表头的 TSV。

## 失败策略

- 优先输出部分真实结果，而不是伪完整结果。
- 如果某个平台受阻，依然交付其余平台，并把受阻渠道单独列入 `not_stably_complete` 区块。
- 必须区分：
  - `no_update`
  - `not_found`
  - `not_stably_complete`
  - `mirror_estimate`

## 参考

- 选择采集路径时，读取 [references/channel-strategies.md](references/channel-strategies.md)
- 需要标准列顺序时，读取 [references/output-template.tsv](references/output-template.tsv)
