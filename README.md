# Competitor Social Tracker Skill | 竞品社媒追踪自动化技能

这是一个专为 AI Agent 设计的**竞品社交媒体内容追踪与分析技能**。它能够自动化地从主流视频平台（YouTube、TikTok、Bilibili）收集指定账号在特定时间窗内的视频动态，进行内容摘要、翻译，并最终导出为结构化的 Excel 报表。

---

## 🚀 核心特性

- **多平台适配**：内置针对 YouTube (静态提取)、TikTok (浏览器模拟)、Bilibili (投稿页抓取) 的差异化采集策略。
- **全自动报表**：从数据采集到摘要生成，再到最终 `.xlsx` 文件导出，实现全链路自动化。
- **内容深度理解**：
  - **多语言处理**：自动提供视频预览标题的中文翻译。
  - **物料提取**：从标题和简介中自动识别皮肤、饰品、套装等核心物料关键词。
  - **风向分析**：基于公开评论互动数据，快速概括评论区情绪趋势。
- **工程化交付**：采用 TSV 作为中间交互格式，确保数据的稳定性和可验证性。

---

## 📂 项目结构

```text
.
├── agents/             # Agent 提示词模板 (OpenAI/Claude 兼容格式)
├── references/         # 平台策略手册与数据输出标准模板
│   ├── channel-strategies.md   # 各平台详细抓取与反爬避让策略
│   └── output-template.tsv     # 标准输出字段顺序定义
├── scripts/            # 后处理脚本
│   └── export_competitor_xlsx.py  # 将 TSV 数据转换为格式化 Excel
├── SKILL.zh-CN.md      # 技能定义核心文档 (中文版)
└── SKILL.md            # 技能定义核心文档 (英文版)
```

---

## 🛠️ 环境要求

- **Python 3.8+**
- 导出脚本所需 Python 依赖：
  ```bash
  pip install pandas openpyxl
  ```
- **Agent 环境**：建议配合具备 `web-access` (浏览器访问) 能力的 AI Agent 使用。

---

## 📝 快速开始

### 1. 技能调用
直接向 Agent 下达指令：
> “帮我追踪最近一周 YouTube 上 @FreeFire 正式频道的视频更新，分析评论风向并导出 Excel。”

### 2. 手动导出报表
如果你已经有了 `competitor-tracker.tsv` 文件，可以手动运行：
```powershell
python "scripts/export_competitor_xlsx.py" `
  --input "路径/to/your.tsv" `
  --output "路径/to/save.xlsx"
```

---

## 📊 输出字段定义

技能默认输出包含以下字段的报表：
- `channel`: 来源渠道
- `account_name`: 账号名称
- `publish_date`: 发布日期
- `source_title`: 原始标题
- `translated_title_zh`: 中文翻译标题
- `video_link`: 视频链接
- `item_or_topic`: 关键物料或讨论主题
- `summary`: 内容简要说明
- `views`: 播放量（原始单位）
- `likes_comments`: 互动量数据
- `comment_direction`: 评论区主要风向/情绪
- `source_note`: 数据来源质量备注

---

## ⚖️ 许可与说明

本工具仅用于公开数据的合规研究与分析。使用时请遵守相关平台的开发者协议与服务条款。
