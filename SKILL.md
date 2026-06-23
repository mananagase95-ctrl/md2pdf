---
name: md-to-pdf
description: "Convert a Markdown spec doc (typically Chinese functional spec with embedded screenshots) to a styled PDF using Edge headless. Use when the user says 生成 PDF / 导出 PDF / 转 PDF / 出一份 PDF on a .md file. Produces a clean, opinionated style — no headers/footers, no TOC, modest image sizing, bold for 「UI/按钮」 strings."
---

# md-to-pdf

Convert a Markdown source file into a styled PDF using Microsoft Edge headless. The visual style is opinionated and stable — do not invent variations.

## When to use

Trigger phrases:
- 「生成 PDF」 / 「导出 PDF」 / 「转 PDF」 / 「出一份 PDF」
- 「重新导出 xxx.pdf」 / 「重新生成 PDF」
- "Generate PDF from this markdown" / "Export to PDF"

Typical inputs: functional spec docs (`.md`) under `~/Desktop/` or `~/Documents/Codex/<date>/.../outputs/`, with embedded screenshots referenced by relative paths.

## When NOT to use

- The user wants a different style (cover page, TOC, page numbers, headers/footers) — ask first, don't invent.
- The source isn't markdown (e.g. they want a PDF from a Google Doc) — use a different approach.
- The user is asking about PDF parsing/reading, not generation.

## Workflow

1. **Read the source `.md`** — extract title, body, image paths, and tables. Image references are relative.
2. **Write intermediate HTML** as `<source-name>_style.html` **in the same directory as the source `.md`** (so relative image paths resolve). Use the template in `## HTML template` below.
3. **Render to PDF via Edge headless**:
   ```bash
   "/Applications/Microsoft Edge.app/Contents/MacOS/Microsoft Edge" \
     --headless --disable-gpu --no-pdf-header-footer \
     --print-to-pdf="<output>.pdf" \
     "file://<absolute path to html>"
   ```
4. **Delete the intermediate `_style.html`** after generation.
5. **Output location**: PDF lands next to the source by default. The user usually wants it on `~/Desktop/`. If the source is already on Desktop, write the PDF there directly; otherwise either write straight to `~/Desktop/<name>.pdf` (preferred, saves a step) or write next to the source and tell the user the path so they can ask to move it.

## Content conversion rules

| Markdown source | HTML output |
| --- | --- |
| `# Title` (first) | `<h1 class="doc-title">` + `<div class="doc-meta">功能说明文档 · YYYY.MM.DD</div>` (date from `# currentDate`) |
| `## Section` | `<h2>` |
| `### Subsection` | `<h3>` |
| `#### Sub-subsection` | `<h4>` |
| `**bold**` | `<strong>` |
| `「UI / 按钮 / 提示」` | wrap in `<strong>` to render as bold dark — **do NOT** use yellow background. This is the standard, reconfirmed 2026-06-23 against 大陆神像展示优化.pdf. |
| `` `code` `` | `<code>` (gray bg, monospace) |
| `\\server\path\...` or filesystem paths shown as content | `<div class="path-block">` (gray bg block, monospace, word-break) |
| `![alt](file.png)` | `<div class="img-single"><figure><img src="file.png"><figcaption>图 N · 简短说明</figcaption></figure></div>` — caption auto-generated from the surrounding heading/context |
| Tables | Plain `<table>` — styled via CSS |

## Don'ts (from explicit user corrections)

- ❌ No special underlines, color bars, or left-border decorations on titles
- ❌ No image borders / rounded corners / padding on `<img>` itself
- ❌ No emoji in the PDF output unless the source has them
- ❌ No page headers / footers / page numbers
- ❌ No Table of Contents / cover page (unless user explicitly asks)
- ❌ **No yellow `.quote` background** for 「」 strings — use `<strong>` instead (this was the old style; deprecated as of 2026-06-23)
- ❌ Don't invent new component styles for one-off needs — reuse `<strong>`, `<code>`, `.path-block`, tables

## HTML template

Use this as the boilerplate. Replace `{{TITLE}}`, `{{DATE}}` (format `YYYY.MM.DD`), and `{{BODY}}`. Keep all CSS verbatim.

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<title>{{TITLE}}</title>
<style>
  @page {
    size: A4;
    margin: 18mm 16mm 18mm 16mm;
  }
  html, body { margin: 0; padding: 0; }
  body {
    font-family: "PingFang SC", "Hiragino Sans GB", "Microsoft YaHei", -apple-system, BlinkMacSystemFont, "Helvetica Neue", sans-serif;
    font-size: 11pt;
    line-height: 1.72;
    color: #475569;
    -webkit-font-smoothing: antialiased;
  }
  strong { color: #1e293b; font-weight: 600; }

  .doc-title {
    font-size: 22pt;
    font-weight: 600;
    color: #1a202c;
    margin: 0 0 6px 0;
    line-height: 1.3;
  }
  .doc-meta {
    font-size: 9pt;
    color: #94a3b8;
    margin: 0 0 22px 0;
  }

  h2 { font-size: 14pt; font-weight: 600; color: #1e293b; margin: 22px 0 10px 0; page-break-after: avoid; }
  h3 { font-size: 12pt; font-weight: 600; color: #334155; margin: 18px 0 8px 0; page-break-after: avoid; }
  h4 { font-size: 11pt; font-weight: 600; color: #475569; margin: 14px 0 6px 0; page-break-after: avoid; }

  p { margin: 6px 0; }
  ul, ol { padding-left: 22px; margin: 6px 0; }
  li { margin: 4px 0; }
  li > p { margin: 2px 0; }

  /* Images — modest, never split across pages */
  .img-single {
    margin: 12px 0;
    text-align: center;
    page-break-inside: avoid;
    break-inside: avoid;
  }
  figure {
    margin: 0;
    page-break-inside: avoid;
    break-inside: avoid;
    display: inline-block;
    max-width: 100%;
  }
  .img-single img {
    max-width: 62%;
    max-height: 250px;
    object-fit: contain;
    display: block;
    margin: 0 auto;
  }
  .img-row {
    display: flex;
    gap: 12px;
    page-break-inside: avoid;
    break-inside: avoid;
    margin: 12px 0;
  }
  .img-row figure { flex: 1; }
  .img-row img {
    width: 100%;
    max-height: 200px;
    object-fit: contain;
    display: block;
  }
  figcaption {
    font-size: 8.5pt;
    color: #94a3b8;
    margin-top: 6px;
    text-align: center;
  }

  /* Inline code & path block */
  code {
    font-family: "SF Mono", Menlo, Consolas, monospace;
    font-size: 9.5pt;
    color: #334155;
    background: #f5f7fa;
    padding: 1px 6px;
    border-radius: 3px;
  }
  .path-block {
    font-family: "SF Mono", Menlo, Consolas, monospace;
    font-size: 9.5pt;
    color: #334155;
    background: #f5f7fa;
    padding: 8px 12px;
    border-radius: 4px;
    word-break: break-all;
    margin: 8px 0 14px 0;
  }

  /* Tables */
  table {
    width: 100%;
    border-collapse: collapse;
    margin: 10px 0 14px 0;
    font-size: 10pt;
    page-break-inside: auto;
  }
  thead { display: table-header-group; }
  tr { page-break-inside: avoid; break-inside: avoid; }
  th, td {
    border: 1px solid #e2e8f0;
    padding: 6px 10px;
    text-align: left;
    vertical-align: top;
    color: #475569;
  }
  th { background: #f5f7fa; color: #1e293b; font-weight: 600; }
  td code { background: #f5f7fa; padding: 1px 5px; }

  /* Optional callout — for BUFF / 投放 / structured info */
  .callout {
    background: #f5f7fa;
    padding: 10px 14px;
    border-radius: 4px;
    margin: 10px 0;
  }
  .callout-label {
    font-weight: 700;
    color: #4f7cff;
    margin-right: 6px;
  }
</style>
</head>
<body>

<h1 class="doc-title">{{TITLE}}</h1>
<div class="doc-meta">功能说明文档 · {{DATE}}</div>

{{BODY}}

</body>
</html>
```

## Example invocation

User says: `生成PDF` on `~/Desktop/某需求文档.md`

```bash
# 1. Read source, build HTML following conversion rules above
# 2. Write _style.html
Write ~/Desktop/某需求文档_style.html

# 3. Render
"/Applications/Microsoft Edge.app/Contents/MacOS/Microsoft Edge" \
  --headless --disable-gpu --no-pdf-header-footer \
  --print-to-pdf="/Users/timesleaper/Desktop/某需求文档.pdf" \
  "file:///Users/timesleaper/Desktop/某需求文档_style.html"

# 4. Cleanup
rm ~/Desktop/某需求文档_style.html
```

Reference output for the visual style: `~/Desktop/大陆神像展示优化.pdf` and `~/Desktop/装备预设.pdf`.
