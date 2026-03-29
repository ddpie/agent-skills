# PPT Generator Skill

专业级演示文稿生成引擎。直接生成 SVG 页面 + svg_to_pptx 转换为原生可编辑 PPTX。
内置 20 种风格模板（ppt-master MIT 许可证），支持深色/浅色/咨询/科技等多种风格。

## 使用场景

- 用户要求生成 PPT / 演示文稿 / 幻灯片
- 用户提供了演示内容、大纲或数据，需要转换为 PPTX
- 用户提到 "做个PPT"、"生成幻灯片"、"演示文稿" 等关键词

## 交互流程（必须遵循）

收到 PPT 需求后，**不要直接生成**，按以下步骤逐步引导用户确认：

### Step 1：确认主题和受众

先理解用户需求，确认关键信息：

> 收到，我来帮你做这个 PPT。先确认几个信息：
> 1. **主题**：{从用户消息提取}，对吗？
> 2. **受众**：给谁看的？（客户/老板/团队/公开演讲）
> 3. **时长**：大概讲多久？（决定页数）
> 4. **有没有必须包含的内容或数据？**

如果用户已经给了足够信息（比如明确说了主题、给了大纲），可以跳过已知项，只问缺失的。

### Step 2：选择风格

把风格选项列给用户，附带简短描述：

> 选个风格吧：
>
> 1️⃣ **dark_warm**（默认）— 深色封面 + 浅色内容页，AI/科技感
> 2️⃣ **麦肯锡** — 白底蓝色，经典咨询风，适合战略分析
> 3️⃣ **Exhibit** — 浅色展示风，适合数据图表密集型
> 4️⃣ **AI Ops** — 全深色，DevOps/AI 运维风
> 5️⃣ **科技蓝商务** — 蓝色科技风，适合商务场景
> 6️⃣ **Smart Red** — 红色商务风
> 7️⃣ **像素复古** — 像素风，游戏/趣味主题
> 8️⃣ **学术答辩** — 浅色简洁，适合论文答辩
> 9️⃣ **政府蓝** — 蓝色庄重风，适合政府报告
> 🔟 **政府红** — 红色党政风
>
> 直接回数字就行，或者描述你想要的感觉我来推荐。

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

### Phase 4：Review（可选）

生成初稿后，可执行审查-修复循环。

**标准 Review（默认）**：至少 2 轮
- Round 1（全量）：演讲教练 + 目标听众 + 领域专家 + 内容审核 + 视觉检查
- Round 2（回归）：视觉 + 内容 + 领域专家
- 退出条件：🔴=0 且 ⚠️≤3，或 round≥4

**快速模式**：用户说"快速出一版"或"不用 review"时跳过。

### Phase 5：交付

只发最终版给用户，中间版本不发（除非用户要求看过程）。

---

## 风格目录映射

| # | 风格名 | 目录名 | 背景 |
|---|--------|--------|------|
| 1 | 深色暖调 | dark_warm | 深色封面+浅色内容 |
| 2 | 咨询风 | consultant | 白底蓝色 |
| 3 | Exhibit | exhibit | 浅色 |
| 4 | AI Ops | ai_ops | 深色 |
| 5 | 科技蓝商务 | 科技蓝商务 | 蓝色 |
| 6 | Smart Red | smart_red | 红色 |
| 7 | 像素复古 | pixel_retro | 深色 |
| 8 | 学术答辩 | academic_defense | 浅色 |
| 9 | 政府蓝 | government_blue | 蓝色 |
| 10 | 政府红 | government_red | 红色 |

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

## 备用引擎（快速生成）

当不需要高视觉质量、只需快速出稿时，可用 template_engine.py：

```python
from template_engine import generate_ppt
slides = [{"type": "cover", "title": "标题", ...}, ...]
generate_ppt(slides, "/tmp/output.pptx", theme="dark_tech")
```

支持 9 种主题：dark_tech（默认）、dark_blue、dark_purple、dark_red、dark_green、dark_warm、consultant、light_corporate、cloud_orange

支持 8 种布局：cover、section、content、table、two_column、stats、cards、closing

---

## 经验教训

### 数据准确性
1. **所有计算必须可验证** — "差 X 倍"必须用原始数据反算验证
2. **比较必须同维度** — 不能拿输入价和输出价比较
3. **对比类 PPT 必须覆盖所有对比对象**
4. **场景选型中推荐的模型必须在前文出现过**
5. **事实数据跨页必须一致**

### 内容结构
6. **章节分隔页必须有过渡句**
7. **Insight 页精简到 3 条以内**
8. **结尾页不能只写"谢谢"** — 必须有 CTA 或核心 takeaway
9. **连续数据表之间需要"呼吸页"**
10. **标题写洞察结论，不写品类标签**

### 视觉排版
11. **深色主题用预混合实色**（#2A4040）而非 alpha 透明度
12. **不用 LibreOffice 验证** — 渲染不可靠，用 PDF→PNG 管线
13. **SVG→PPTX 管线优先** — 视觉质量远超 python-pptx
14. **每页必须饱满** — 3 条 bullet 是 content 页下限

### 流程
15. **多轮 Review 是必须的** — 单轮不够
16. **只发最终版** — 中间版本不发给用户
