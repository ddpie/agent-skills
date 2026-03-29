# ppt-generator

AI agent skill for generating presentation slides. Converts structured content into polished PPTX files via SVG intermediate format.

AI Agent 技能，用于生成演示文稿。通过 SVG 中间格式将结构化内容转换为精美的 PPTX 文件。

## Features / 功能

- 9 color themes / 9 种配色主题
- 8 layout templates (cover, chapter, content, ending) / 8 套布局模板
- 30+ chart templates (bar, line, pie, funnel, gantt, SWOT, etc.) / 30+ 图表模板
- 600+ icons (downloaded on-demand from SVG Repo) / 600+ 图标（按需下载）
- python-pptx native engine / python-pptx 原生引擎

## Themes / 主题

| Theme | Style | 风格 |
|-------|-------|------|
| dark_tech | Dark tech (default) | 深色科技（默认） |
| dark_warm | Dark warm tone | 深色暖调 |
| dark_blue / purple / red / green | Dark color variants | 深色变体 |
| consultant | White + blue, consulting | 咨询风 |
| light_corporate | Light corporate | 浅色商务 |
| cloud_orange | Deep navy + orange | 深蓝+橙色 |

## Layouts / 布局模板

| Layout | Style | 风格 |
|--------|-------|------|
| dark_warm | AI / tech | AI/科技 |
| consultant | Strategy analysis | 战略分析 |
| cloud_orange | Cloud architecture | 云架构 |
| ai_ops | AI operations | AI 运维 |
| exhibit | Data showcase | 数据展示 |
| smart_red | General business | 通用商务 |
| pixel_retro | Creative / fun | 创意趣味 |
| 科技蓝商务 | Formal business | 正式商务 |

## Setup / 安装

```bash
pip install python-pptx Pillow
python3 ppt-master-assets/scripts/download_icons.py  # optional / 可选，图标支持
```

## License / 许可

MIT
