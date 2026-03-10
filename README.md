# wecomdocs-skill

Claude Code / Cursor plugin for operating WeCom Docs (企业微信文档) spreadsheets via `wecomdocs-cli`.

企业微信文档表格操作的 AI 编程助手插件，通过 `wecomdocs-cli` 实现对企微在线表格的读写操作。支持 Claude Code 和 Cursor。

## Installation / 安装

### Claude Code

在 Claude Code 对话中输入：

```
/plugin marketplace add https://github.com/Hallalei/wecomdocs-skill.git
/plugin install wecomdocs-skill
```

或通过 CLI：

```bash
claude plugin marketplace add https://github.com/Hallalei/wecomdocs-skill.git
claude plugin install wecomdocs-skill
```

更新：

```
/plugin update wecomdocs-skill
```

### Cursor

将 skill 复制到 Cursor skills 目录即可：

```bash
# 全局安装（所有项目可用）
mkdir -p ~/.cursor/skills/wecomdocs-skill
curl -o ~/.cursor/skills/wecomdocs-skill/SKILL.md https://raw.githubusercontent.com/Hallalei/wecomdocs-skill/main/skills/wecomdocs-skill/SKILL.md

# 或仅当前项目
mkdir -p .cursor/skills/wecomdocs-skill
curl -o .cursor/skills/wecomdocs-skill/SKILL.md https://raw.githubusercontent.com/Hallalei/wecomdocs-skill/main/skills/wecomdocs-skill/SKILL.md
```

更新时重新执行上述命令即可。

## Prerequisites / 前置依赖

- [wecomdocs-cli](https://www.npmjs.com/package/wecomdocs-cli) (v0.6.x+)
  ```bash
  npm i -g wecomdocs-cli
  ```
- [Playwright](https://playwright.dev/) (for login)
  ```bash
  npx playwright install chromium
  ```

## Features / 功能

- **Document browsing** - List recent docs, browse shared spaces, search by title
- **Spreadsheet reading** - Read sheets with tab selection, row filtering, merged cell handling
- **Spreadsheet writing** - Write to cells with safety confirmation workflow
- **Edit history** - View edit history and version diffs
- **Smart output** - Role-based output formatting (PM / Engineer / TL)
- **User identity** - Persistent user profile for personalized queries

## Usage Examples / 使用示例

安装后，在 Claude Code 或 Cursor 中直接对话：

- "帮我看一下最近的企微文档"
- "读取这个表格 e3_xxx 的数据"
- "搜索标题包含'周报'的文档"
- "把 A1 单元格改成'已完成'"
- "我需要关注哪些任务？"

## License

MIT
