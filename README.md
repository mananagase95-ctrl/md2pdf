# md2pdf

A [Claude Code](https://claude.com/claude-code) skill that converts a Markdown spec document (typically a Chinese functional spec with embedded screenshots) into a styled PDF using Microsoft Edge headless rendering.

## What it does

When you say things like `生成 PDF` / `导出 PDF` / `转 PDF` on a `.md` file, Claude:

1. Reads the source Markdown.
2. Generates an intermediate HTML using a clean, opinionated stylesheet (A4, PingFang SC, slate-gray body, no headers/footers, no TOC, modest image sizing, bold for `「UI/按钮」` strings).
3. Renders to PDF via headless Edge.
4. Cleans up the intermediate HTML.

The style is tuned for game / product functional spec documents with Chinese content and embedded screenshots, but the template is general enough for any spec-style document.

## Install

Place [SKILL.md](SKILL.md) at:

```
~/.claude/skills/md-to-pdf/SKILL.md
```

That's it. Claude Code auto-discovers skills under `~/.claude/skills/<name>/SKILL.md` and surfaces them based on the `description` in the frontmatter.

## Requirements

- macOS with Microsoft Edge installed at `/Applications/Microsoft Edge.app`
- Claude Code

Edge is used because its headless `--print-to-pdf` mode reliably renders CJK fonts and supports the CSS used in the template (`page-break-inside: avoid`, `@page` margins, etc.) without extra config. You can adapt the toolchain to Chrome by changing the binary path in the workflow.

## Style spec

See [SKILL.md](SKILL.md) for the full template, conversion rules, and "Don'ts" list.

## License

MIT
