---
name: ppt-generator
description: SVG-based PPT generator with 9 themes, 8 layouts, 30+ charts, and 600+ icons
---

# PPT Generator Skill

专业级演示文稿生成引擎。直接生成 SVG 页面 + svg_to_pptx 转换为原生可编辑 PPTX。
内置 8 套布局模板，支持深色/浅色/咨询/科技等多种风格。

## 使用场景

- 用户要求生成 PPT / 演示文稿 / 幻灯片
- 用户提供了演示内容、大纲或数据，需要转换为 PPTX
- 用户提到 "做个PPT"、"生成幻灯片"、"演示文稿" 等关键词

## 交互流程（必须遵循）

收到 PPT 需求后，**不要直接生成**，按以下步骤逐步引导用户确认：

### Step 1：确认主题

先确认用户要做什么：

> 收到，我来帮你做这个 PPT。主题是「{从用户消息提取}」对吗？

如果用户已经给了详细大纲，直接跳到 Step 2。

### Step 2：选择风格

> 选个风格（回数字就行）：
>
> 1️⃣ **dark_warm**（默认）— 深色暖调，AI/科技感
> 2️⃣ **consultant** — 白底蓝色，咨询风
> 3️⃣ **cloud_orange** — 深蓝+橙色，云/技术架构
> 4️⃣ **ai_ops** — 全深色，运维风
> 5️⃣ **科技蓝商务** — 蓝色科技，正式商务
> 6️⃣ **smart_red** — 红色商务
> 7️⃣ **exhibit** — 浅色展示，数据密集型
> 8️⃣ **pixel_retro** — 像素复古，创意趣味

### Step 3：确认大纲结构

根据主题和风格，给出建议大纲让用户确认：

> 根据你的需求，我建议这样的结构（共 {N} 页）：
>
> 📄 P1 — 封面：{标题}
> 📄 P2 — 目录/概览
> 📄 P3 — {章节1标题}
> ...
> 📄 P{N} — 结束页：{核心 takeaway / CTA}
>
> 要调整哪里？还是直接开始生成？

### Step 4：生成

用户确认后才开始生成。生成过程中给用户一条状态消息：

> 开始生成，预计 X 分钟。

然后执行下方「技术流程」。

### Step 5：交付和迭代

发送最终版 PPTX，附带简要说明：

> PPT 已生成，共 {N} 页。
> 需要调整的话直接说，比如：
> - "第3页数据换成最新的"
> - "加一页关于 XX 的内容"
> - "整体颜色再深一点"

---

## 技术流程（Step 4 内部执行）

```
读 design_spec.md + 参考 SVG → 直接写 SVG 文件 → svg_to_pptx 转 PPTX → 交付
```

### Phase 1：读取设计规范

读取目标风格的设计规范和参考模板：

```
ppt-master-assets/templates/layouts/{风格}/design_spec.md   ← 配色、字体、布局规范
ppt-master-assets/templates/layouts/{风格}/01_cover.svg      ← 封面参考
ppt-master-assets/templates/layouts/{风格}/02_chapter.svg    ← 章节页参考
ppt-master-assets/templates/layouts/{风格}/03_content.svg    ← 内容页参考
ppt-master-assets/templates/layouts/{风格}/04_ending.svg     ← 结束页参考
```

### Phase 2：直接生成 SVG 文件

用 `write` 工具逐页生成 SVG 文件到 `/tmp/ppt_svgs/{风格}/` 目录。

**SVG 关键规则：**
1. `viewBox` 必须是 `0 0 1280 720`（16:9）
2. 严格遵循 design_spec.md 的配色、字体、布局规范
3. **禁止使用**：foreignObject、clipPath、mask、`<style>`、class
4. 文件命名：`01_cover.svg`、`02_toc.svg`、`03_chapter1.svg`... 按顺序排列
5. 所有文字直接用 `<text>` 元素，设置 `font-family`、`font-size`、`fill`
6. 背景用 `<rect>` 全覆盖，装饰用 `<rect>`/`<circle>`/`<line>`/`<path>`
7. 表格用 `<rect>` + `<text>` 手动排列（不用 HTML table）
8. 排版饱满，不留大片空白
9. 标题写洞察结论，不写品类标签

**生成顺序建议：**
- 先写封面和结束页（定调）
- 再写章节分隔页（统一风格）
- 最后写内容页（数据密集，逐页处理）

### Phase 3：SVG → PPTX 转换

svg_to_pptx 将 SVG 元素转为**原生 DrawingML 形状**（不是图片）：
- `<text>` → 可编辑文本框（双击即可改文字）
- `<rect>` → 原生矩形（可拖拽、改色）
- `<circle>` / `<ellipse>` → 原生圆形
- `<line>` / `<path>` → 原生线条/路径
- 表格（rect + text 组合）→ 可编辑的形状组

生成的 PPTX 和手动做的一样，用户可以直接在 PowerPoint 里编辑所有内容。

```python
import sys, os
sys.path.insert(0, os.path.expanduser(
    "~/.openclaw/workspace/skills/ppt-generator/ppt-master-assets/scripts"))
from svg_to_pptx import create_pptx_with_native_svg
from pathlib import Path

svgs = sorted(Path("/tmp/ppt_svgs/{风格}").glob("*.svg"))
create_pptx_with_native_svg(svgs, Path("/tmp/openclaw/output.pptx"),
                            canvas_format="ppt169",
                            use_native_shapes=True,  # 必须！否则只是嵌入 SVG 图片
                            verbose=True)
```

### Phase 4：交叉 Review（可选，需用户确认）

生成初稿后，询问用户是否需要多轮审查：

> PPT 初稿已生成。要不要跑一轮交叉 Review？多个 reviewer 并行检查，会更精细但需要几分钟。

如果用户同意且 agent 支持 subagent，执行审查-修复循环：

1. **Round 1（全量审查）**：并行启动 5 个 reviewer subagent
   - 🎤 演讲教练 — 叙事弧线、信息流、节奏
   - 👥 目标听众 — 模拟受众反应、理解难度
   - 🔬 领域专家 — 事实准确性、技术深度
   - 📋 内容审核 — 结构规范、错别字、数据一致性
   - 👁️ 视觉检查 — 排版、对齐、可读性

2. **修复**：汇总所有 reviewer 的问题，按优先级修复（🔴 先于 ⚠️）

3. **Round 2（回归验证）**：3 个 reviewer 回归检查修复结果

4. **退出条件**：🔴=0 且 ⚠️≤3，或 round≥4 强制退出

用户说"不用 review"或"快速出一版"时跳过。

### Phase 5：交付

只发最终版给用户，中间版本不发（除非用户要求看过程）。

---

## 风格目录映射

| # | 风格名 | 目录名 | 背景 |
|---|--------|--------|------|
| 1 | 深色暖调 | dark_warm | 深色封面+浅色内容 |
| 2 | 咨询风 | consultant | 白底蓝色 |
| 3 | 云橙 | cloud_orange | 深蓝+橙色 |
| 4 | AI Ops | ai_ops | 深色 |
| 5 | 科技蓝商务 | 科技蓝商务 | 蓝色 |
| 6 | Smart Red | smart_red | 红色 |
| 7 | Exhibit | exhibit | 浅色 |
| 8 | 像素复古 | pixel_retro | 深色 |

用户偏好深色背景，默认推荐 dark_warm。

### 设计规范位置

每套模板在 `ppt-master-assets/templates/layouts/{目录名}/` 下包含：
- `design_spec.md` — 完整设计参数
- `01_cover.svg` — 封面模板
- `02_chapter.svg` / `02_toc.svg` — 章节/目录页
- `03_content.svg` — 内容页
- `04_ending.svg` — 结束页

### 参考角色文档

`ppt-master-assets/references/` 下的关键文档：
- `strategist.md` — 策略师角色
- `executor-consultant-top.md` — 顶级咨询风执行器
- `shared-standards.md` — SVG 技术约束

---
