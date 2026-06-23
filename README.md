# md2pdf

一个把 Markdown 需求文档（中文功能说明文档，含截图）渲染成统一样式 PDF 的 Agent 技能。基于 macOS Edge headless 渲染，无需外部依赖。

同时支持 **Claude Code** 和 **OpenAI Codex CLI**——两边的 Skill 加载约定都是 `<skills 根目录>/<技能名>/SKILL.md`，所以同一份 [SKILL.md](SKILL.md) 可以被两个工具复用。

## 这个技能能做什么

当你对一份 `.md` 文档说「生成 PDF」「导出 PDF」「转 PDF」「出一份 PDF」时，AI 会：

1. 读取源 Markdown，解析标题、正文、图片引用、表格等。
2. 按照内置的样式规范生成中间 HTML（A4、PingFang SC、灰蓝正文、`「UI / 按钮」`一律加粗、图片不跨页、无页眉页脚、无 TOC）。
3. 用 macOS Microsoft Edge headless 渲染成 PDF。
4. 清理中间 HTML。

输出风格针对**游戏 / 产品功能说明文档**优化，但模板本身对任意中文规范类文档同样适用。

## 安装

只需要把 [SKILL.md](SKILL.md) 放到对应工具的 skills 目录下即可。

### Claude Code

```bash
mkdir -p ~/.claude/skills/md-to-pdf
curl -fsSL https://raw.githubusercontent.com/mananagase95-ctrl/md2pdf/main/SKILL.md \
  -o ~/.claude/skills/md-to-pdf/SKILL.md
```

或者直接 clone：

```bash
git clone https://github.com/mananagase95-ctrl/md2pdf.git ~/.claude/skills/md-to-pdf
```

### OpenAI Codex CLI

Codex CLI 从 2025-12 起原生支持 Skills，约定路径为 `~/.codex/skills/<name>/SKILL.md`：

```bash
mkdir -p ~/.codex/skills/md-to-pdf
curl -fsSL https://raw.githubusercontent.com/mananagase95-ctrl/md2pdf/main/SKILL.md \
  -o ~/.codex/skills/md-to-pdf/SKILL.md
```

或 clone：

```bash
git clone https://github.com/mananagase95-ctrl/md2pdf.git ~/.codex/skills/md-to-pdf
```

### 同时供 Claude Code 和 Codex 使用（推荐：软链）

把真实文件放在一个地方，从另一边软链过去，避免维护两份副本：

```bash
# 真实文件放在 Claude 目录
git clone https://github.com/mananagase95-ctrl/md2pdf.git ~/.claude/skills/md-to-pdf

# Codex 端做软链
ln -s ../../.claude/skills/md-to-pdf ~/.codex/skills/md-to-pdf
```

## 触发方式

安装后，对任意 `.md` 文件说下面任一句话即可：

- 「生成 PDF」「导出 PDF」「转 PDF」「出一份 PDF」
- 「重新导出 xxx.pdf」「按统一样式重新生成 PDF」
- 英文也可以：`Generate PDF from this markdown` / `Export to PDF`

AI 会自动按 SKILL.md 中的工作流和样式规范执行。

## 运行要求

- macOS
- 已安装 Microsoft Edge：`/Applications/Microsoft Edge.app`
- Claude Code 或 OpenAI Codex CLI

之所以选 Edge 而不是其他浏览器：Edge 的 headless `--print-to-pdf` 模式对中日韩字体渲染最稳定，且原生支持模板里用到的 `page-break-inside: avoid`、`@page` 自定义页边距等 CSS，无需额外配置。想换成 Chrome 也只需替换 SKILL.md 工作流中的可执行文件路径。

## 样式细节

完整的 HTML / CSS 模板、Markdown → HTML 转换规则、以及"不要做什么"清单都写在 [SKILL.md](SKILL.md) 里。简单说：

- **页面**：A4，页边距 `18mm 16mm 18mm 16mm`，无页眉页脚
- **字体**：`PingFang SC` 优先，正文 11pt / 行高 1.72 / 颜色 `#475569`
- **标题**：22pt 深色，无下划线 / 边框装饰
- **`「UI/按钮」`字段**：一律 `<strong>` 加粗深色（不用黄色背景高亮）
- **行内代码 / 字段 KEY**：等宽字体 + 浅灰背景
- **图片**：`max-width: 62%`，`max-height: 250px`，强制不跨页

## 许可证

MIT
