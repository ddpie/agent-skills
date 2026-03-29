# ppt-generator

AI agent skill for generating presentation slides. Converts structured content into polished PPTX files via SVG intermediate format.

## Features

- 9 color themes (dark_tech, dark_warm, consultant, cloud_orange, etc.)
- 8 layout templates with cover, chapter, content, and ending pages
- 30+ chart templates (bar, line, pie, funnel, gantt, SWOT, etc.)
- 600+ icons (downloaded on-demand from SVG Repo)
- python-pptx native engine for direct PPTX generation

## Themes

| Theme | Style |
|-------|-------|
| dark_tech | Dark tech (default) |
| dark_warm | Dark warm tone |
| dark_blue / dark_purple / dark_red / dark_green | Dark color variants |
| consultant | White + blue, consulting style |
| light_corporate | Light corporate |
| cloud_orange | Deep navy + orange |

## Setup

```bash
pip install python-pptx Pillow
python3 ppt-master-assets/scripts/download_icons.py  # optional, for icon support
```

## License

MIT (based on [ppt-master](https://github.com/hugohe3/ppt-master))
