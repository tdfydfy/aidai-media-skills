---
name: aidai-ppt-skill
description: "Four-stage content-to-deliverable pipeline: extract structure from long-form content → humanize to bilingual speaker-ready copy → confirm with user → render as final output (slide deck / PPT / HTML / video script / infographic)"
version: 1.1.0
author: hexiaoma
license: MIT
platforms: [linux, macos, windows]
tags: [presentation, pipeline, content-to-slides, humanizer-zh, frontend-slides, workflow, ppt, video-script]
---

# AI智有方 PPT 流水线 (aidai-ppt-skill)

Orchestrate the four-stage content-to-deliverable pipeline:

```
Source Document → [content-to-slides] → [humanizer-zh] → [内容确认] → [渲染/输出]
  (long-form text)   (extract structure)  (bilingual copy)  (用户审核文案)  (最终交付物)
```

**适用输出形式：**
- HTML 幻灯片（16:9 单文件，← → 翻页）
- 传统 PPT（大纲+文案，可交给 Keynote/PPT 制作）
- 视频脚本（旁白 + 画面描述 + 节奏标记）
- 信息图文案（信息层级 + 文案落地）
- 演讲提词稿（speaker-ready 逐页稿）
- 公众号/小红书图文排版素材

## When to use this pipeline

Load this skill when the user asks to:
- "turn this document/report/essay into a presentation"
- "make a slide deck from this text"
- "generate an HTML talk from this content"
- "present this topic as a web-based slide show"
- "write a video script from this material"
- "make a project introduction page" (scrolling or slide deck — confirm format first)
- "帮我梳理这篇内容做成PPT"
- "把这个文章改成视频脚本"
- "提炼这个文档做演讲提词稿"

This is a meta-skill that loads and coordinates three sub-skills. Load it FIRST.

## Required sub-skills

| Stage | Skill | Function |
|---|---|---|
| 1 | `content-to-slides` | Extract structural skeleton from source text |
| 2 | `humanizer-zh` | Rewrite to natural Chinese + bilingual speaker copy |
| 3 | `frontend-slides` | Render as single-file 16:9 HTML (one of multiple output options, see below) |

## Pipeline steps

### ⚠️ Step 0a: 骨架确认 — 结构与风格定稿

**Do NOT skip this step.** The user explicitly rejected jumping straight to code without human confirmation (用户原话: "这个过程太自动化了，我建议中间还是需要和人类确认一下：核心观点，章节目录，风格比例等，然后再进行制作").

**这一步只确认骨架（结构、风格、格式），不涉及具体文案内容。** 具体文案有独立确认步骤（见 Step 2.5）。

Before writing a single line of HTML or content:

1. **Propose a structure outline** — Present a draft of slides/sections with titles. Ask the user to confirm or modify.
2. **Confirm format** — 要什么输出形式？
   - Slide deck（HTML 幻灯片，← → 翻页）
   - 传统 PPT（提供结构和文案，手工制作）
   - 视频脚本（旁白 + 画面描述）
   - 信息图/公众号图文
   - 演讲提词稿
   - 其他？
3. **Confirm tone** — Application-oriented (pain → solution → benefits) or technical/feature tour?
4. **Confirm design direction** — Dark theme with full-bg images, or light clean scrolling? English as decoration or full bilingual?
5. **Confirm page count** — Not always 12. The content determines the count. Ask: "这个量, 你觉得几页合适?"
6. **Ask about reference material** — Is there an existing project, README, or asset the page should refer to?

Always present a concrete proposal (2-4 sentences + bullet list of sections), then wait for feedback. A "looks good" or "调整一下" response is the green light to proceed to content generation. If they say "直接做" without confirming anything, push once more with a specific choice ("暗色背景还是白色?" "翻页还是滚动?").

See `references/pre-production-check.md` for the exact template.

### 1. Extract structure → Slide skeleton

Load `content-to-slides` skill. Apply it to the source document:
- Identify narrative arc (hook → core → call)
- Produce N slides based on content, assigned slide types per section
- Each slide: title_cn, title_en, body_cn, body_en
- **IMPORTANT: Not every output needs to be a slide deck. Confirm the format (slides vs scrolling page) with the user before generating.**

Output: JSON array of slide objects, or section outline for scrolling page.

### 2. Humanize copy

Load `humanizer-zh` skill. Apply to each slide's body copy:
- Strip AI-isms and translationese from Chinese text
- Short sentences, natural speaking rhythm
- Chinese primary, English secondary — NOT literal translation, same idea two voices
- Final punchline per slide

Output: Same JSON array with rewritten body_cn and body_en fields.

### ⚠️ Step 2.5: 内容确认 — 用户审核正文文案

**在渲染之前，必须让用户过一遍实际文案内容，确认没问题再渲染。**

骨架确认（Step 0a）只定了标题和结构，正文具体怎么写用户还没有看过。这一步把每页的正文文案（中英双语）完整呈现给用户。

**怎么做：**

1. **整理内容审阅摘要** — 用表格/结构化形式呈现每页的：
   - 页面标题（中文）
   - 正文文案（中文，这是重点——用户主要审阅的部分）
   - 英文文案（供参考，按 slide 类型定程度）
2. **标注需要特别关注的页面** — 比如首页/结语/技术页等容易出问题的
3. **提出问题引导反馈** — 例如：
   - "首页切入点这样写可以吗？"
   - "技术页的表述你确认一下？"
   - "结尾的话术力度够不够？"
   - "整体语气是不是你想要的？"
4. **注意：只呈现文案，不展示设计（背景图/布局/配色）**——避免用户被视觉效果干扰文案判断

**呈现格式示例：**

```
📝 内容确认，请过目：

1️⃣ 标题页
【痛点】你知道XX有多痛吗？
EN: You know how painful XX is?

2️⃣ 方案页  
【方案】三步解决你的XXX
- 第一步：...
- 第二步：...
- 第三步：...
EN: Three steps to fix your XXX

3️⃣ 技术页
【技术】基于XXX架构
...
EN: Powered by XXX

---
请看看文案有没有需要调整的地方？
- 整体语气OK吗？
- 哪页需要改？
- 有什么想去掉或者补充的？
```

**用户确认后**（"OK"、"可以"、"改一下第X页"），按反馈修改后再进入渲染。

**如果用户要求直接渲染：** 只在用户明确说"不用看直接做"或"行就这样"时才跳过此步。任何含糊回应（"嗯"、"好"、"先做吧"）都应追问一次以确认。

### 3. 渲染/输出（按格式定方案）

**输出方式取决于用户在 Step 0a 确认的格式：**

- **HTML 幻灯片** → 加载 `frontend-slides` skill，生成 1920×1080 16:9 单文件 HTML
  - Apple-style minimal design，混合排布（`full-bg` / `split-img` / `half-img` / `deco` / `text-only`）
  - Unsplash 背景图，键盘+点击翻页，进度条/圆点指示器
- **传统 PPT** → 导出结构化文案 + 排版建议，交给 Keynote/PPT 软件制作（文案已确认通过 Step 2.5）
- **视频脚本** → 按画面/场景分镜输出：旁白文案 + 画面描述 + 时长标记
- **信息图/公众号** → 按视觉层级输出文案+配图建议，适配对应平台规格
- **演讲提词稿** → 逐页 speaker-ready 稿，标注节奏/停顿/重点

Output: 按格式准备的最终交付物。HTML 文件 or 结构化文档。

### 4. Deliver

- Return the file path to the user
- For Feishu users, send as file attachment via `MEDIA:/path/to/file.html`
- For web deployment (e.g., show.daiworld.cc.cd), see the `1panel-openresty-deploy` skill — copy to `/opt/1panel/www/sites/<site>/index/` on the host (NOT `/www/sites/<site>/index/` which is often a different directory)

## Pipeline script (reference)

A reusable Python script lives at the `frontend-slides` skill directory. When running the pipeline:

```
python3 ~/.hermes/profiles/hexiaoma/skills/creative/frontend-slides/scripts/gen_slides.py
```

This script:
1. Defines the slide JSON data (from content-to-slides + humanizer-zh)
2. Generates the single-file HTML with background images
3. Outputs to `/root/presentation_output.html`

### Deployment

The Hermes agent runs on the production server directly (ser112051180610). No SSH needed.

```bash
# Generate
python3 skills/creative/frontend-slides/scripts/gen_slides.py

# Deploy
cp /root/presentation_output.html /opt/1panel/www/sites/show/index/presentation.html
chmod 644 /opt/1panel/www/sites/show/index/presentation.html

# Verify
curl -s -o /dev/null -w '%{http_code}' https://show.daiworld.cc.cd/presentation.html
```

The `references/deploy-troubleshoot-checklist.md` file documents the step-by-step debugging workflow for when a deployed file doesn't show up (Docker bind-mount path check → CF cache diagnosis → bypass methods → user communication).

## Pitfalls

- **CRITICAL: Load and use the sub-skills. Do not hand-roll HTML outside the frontend-slides design system.** The user explicitly called this out (用户原话: "你没有使用前面那几个技能吗？怎么感觉前后风格差别很大"). Loading `frontend-slides` and ignoring its design tokens produces inconsistent output.
- **Confirm before generating.** The user rejected the fully automated approach (用户原话: "这个过程太自动化了，需要和人类确认一下核心观点、章节目录、风格比例等"). Always do Step 0a.
- **内容确认不要跳过。** 即使是同一个人写的内容，呈现给用户看后往往会有调整。骨架确认（Step 0a）和内容确认（Step 2.5）是两道独立关卡，缺一不可。
- **Document too short:** If <5 paragraphs, skip the pipeline — a slide deck is overkill
- **Document too dense:** Use the Introduction + Conclusion + strongest chapter only
- **Religious/philosophical text:** Needs extra humanization — strip hedging, citations mid-flow, abstraction chains
- **Technical document:** Skip serif fonts, use system sans throughout
- **Chinese-only content:** Produce English as a summary rather than full translation
- **Font loading offline:** Add a note that Noto Serif SC requires internet on first load; falls back to PingFang SC / system serif
- **Cloudflare cache:** If domain is behind CF proxy, the cached old version won't update. Deploy with a new filename (e.g., `mh.html` instead of `presentation.html`) to bypass cache entirely, or add `?v=N` query param
- **Image layout variety:** Do NOT use `full-bg` for every slide. Mix layouts (`full-bg`, `split-img`, `half-img`, `deco`, `text-only`) based on slide content emotion. Using the same layout for all slides was explicitly rejected by the user (用户原话: "不能所有页面同等的大背景图")

## Deployment considerations

The generated `.html` is a self-contained single file — perfect for serving behind a reverse proxy.

### FIRST RULE: Check existing web infrastructure — do NOT reach for tunnels or new servers

**Before starting any cloudflared, ngrok, Python http.server, reverse proxy script, or nginx config change: look for an existing web server the file can be dropped into.**

The most common deployment is simply:
```bash
cp generated.html /opt/1panel/www/sites/<site>/index/presentation.html
```
That's it. The file is now at `https://<domain>/presentation.html`.

**How to check (in order):**
1. Look at the nginx/OpenResty config for the target domain — does it have a `root` directive under `location /` or a `try_files $uri` fallback?
2. Does `autoindex on` exist? Then anything in the web root is automatically served.
3. If it's a pure `proxy_pass` with no static fallback, THEN consider tunnels or config edits.

The user will (rightfully) call you out if you start building tunnels and proxy scripts when a simple file copy would work.

### Behind 1Panel/OpenResty — static files

**⛔ FIRST, CHECK THE ACTUAL PATH.** This is the #1 cause of "文件放过去但用户看不到" bugs.

```bash
# Check BOTH paths — they are NOT the same directory!
ls -la /www/sites/<site>/index/
ls -la /opt/1panel/www/sites/<site>/index/

# One of them has the existing site files, the other is empty or different.
# The one with existing files is the correct path.
# On 1Panel servers, Docker maps:  /opt/1panel/www  →  /www (inside container)
# So the site config's root /www/sites/... maps to /opt/1panel/www/sites/... on host
```

If the site config has `location / { try_files $uri $uri/ /index.html; }` (or `autoindex on`), just copy the file:

```bash
# Copy to the HOST path that Docker maps — NOT /www/ directly
cp presentation.html /opt/1panel/www/sites/<site>/index/
chmod 644 /opt/1panel/www/sites/<site>/index/presentation.html
```

Then verify the container sees it:
```bash
docker exec <container-id> ls -la /www/sites/<site>/index/presentation.html
```

And verify the content:
```bash
docker exec <container-id> head -5 /www/sites/<site>/index/presentation.html
```

Public verification:
```bash
curl -s -o /dev/null -w '%{http_code}' https://<domain>/presentation.html
```

**⚠️ Critical path distinction**: On 1Panel servers, `/www/` (host-level) and `/opt/1panel/www/` (Docker mount) are usually DIFFERENT directories. The Docker container sees `/opt/1panel/www/` mapped as `/www/`. Always deploy to `/opt/1panel/www/sites/<site>/...`, NOT `/www/sites/<site>/...`. **Always check both paths before copying** — you WILL eventually guess wrong if you don't.

### Cloudflare Tunnel (last resort)

For quick external access when existing infrastructure can't be used:
```bash
cloudflared tunnel --url http://localhost:<port> --protocol http2
```
Uses `*.trycloudflare.com`. Note: if QUIC/UDP outbound is blocked, pass `--protocol http2`.
