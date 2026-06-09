# Pre-Production Checklist: Confirm Before Generating

**Use this when the user asks for any HTML output — slide deck, project page, landing page, etc.**

Present this as a conversation, not a form. Pick the relevant questions and ask 2-4 at a time. Don't dump the entire list.

## When to use this checklist

This is NOT an optional step. The user explicitly rejected skipping it (用户原话: "这个过程太自动化了，我建议中间还是需要和人类确认一下：核心观点，章节目录，风格比例等，然后再进行制作"). Always run through this before writing any HTML.

Use it for both slide decks AND scrolling pages. The format differs but the confirmation process is the same.

## Format

- **Slide deck** — ← → navigation, 1920×1080 canvas, full-screen, 6-12 pages
- **Scrolling page** — one-page scroll, sections with dividers, hero at top
- **Mixed** — first few slides as deck, then scroll for details

## Tone & focus

- **Application-oriented** — pain points → solution → benefits → CTA. Tech consolidated to ONE slide. Good for sales/marketing/intro audience. **Recommended default.**
- **Technical/feature tour** — architecture → features → specs → comparison. Tech distributed. Good for dev/teams audience.
- **Balanced** — front half tells the story, back half shows the tech. Use when the audience includes both decision-makers and technical reviewers.

## Design style

| Dimension | Option A | Option B |
|---|---|---|
| Background | Dark (full-bg images + overlay) | Light (white/gray, no images) |
| Cards | Frosted glass (`backdrop-filter: blur`) | Solid white (`#fff`) |
| English | Moderate — section labels, taglines, key terms | Full bilingual — every block CN+EN |
| Font | Noto Serif SC for headings + SF Pro body | System sans only |
| Highlight color | Blue (`#0066cc` / `#60a5fa`) | Orange/green (project brand color) |

## Page structure (example to modify)

```
Hero → Problem → Solution → How It Works → Benefits → Tech → CTA
```

or

```
Hero → Features (grid) → Architecture → Scoring → Compliance → Footer
```

Adjust based on what needs to be communicated.

## 区分：骨架确认 vs 内容确认

**⚠️ 重要：这个检查清单只负责骨架确认（结构、风格、格式）。** 不要在这里纠结具体文案。

流程现在是两轮确认：
1. **骨架确认（Step 0a）** — 用本清单确认目录结构、风格基调、页数。**此时还没有写具体文案。**
2. **内容确认（Step 2.5）** — 文案写完后，把每页正文完整呈现给用户审阅，确认后再渲染。

**不要在第一轮就抛文案细节**，用户会被淹没。第一轮只谈骨架，第二轮再谈血肉。

## Common mistakes to avoid

- ❌ Jumping straight to code without confirming the outline
- ❌ Using the same format as the last project (every project is different)
- ❌ Mixing dark and light theme slides in one deck — choose ONE
- ❌ Defaulting to 12 slides when 7-8 would be tighter
- ❌ Assuming the user wants the same English level as before
- ❌ Spreading technical details across multiple slides — consolidate into ONE "Architecture" slide for application-oriented decks

## Recommended default: Application-oriented pitch deck

When the user hasn't specified direction, this pattern works for most competition/intro/pitch scenarios:

```
1. Title — dramatic full-bg
2. The Pain — 3 problems or a clear tension statement
3. The Solution — one-sentence callout, the "aha" moment
4. How It Works — steps, flow, or customer story (split-img or 3-card layout)
5. What You Gain — benefits grid or checklist
6. Tech Overview — one slide only, consolidated (architecture + key facts)
7. Compliance / Trust — credentials, security, red lines (if relevant)
8. CTA / Closing — command, link, or next step
```

All slides use the **same theme** (dark or light — pick one). Tech is exactly one slide. The rest is all about the user's problem and your solution.

## Template for asking

```
我先看一下项目/文档，然后给你几个方案让你选。

我的理解：
- 这是一个XX用途的页面
- 核心信息点：A, B, C
- 我建议的结构：XXX

你觉得：
1. 翻页slide deck 还是 滚动页面？
2. 暗色大图 还是 白色干净版？
3. 技术内容多还是应用内容多？
```

Wait for their answer to at least Q1 before proceeding.
