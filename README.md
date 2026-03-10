# wecomdocs-skill

Claude Code plugin for operating WeCom Docs (企业微信文档) spreadsheets via `wecomdocs-cli`.

企业微信文档表格操作的 Claude Code 插件，通过 `wecomdocs-cli` 实现对企微在线表格的读写操作。

## Installation / 安装

```bash
/plugin install https://github.com/Hallalei/wecomdocs-skill
```

To update, re-run the install command.

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

Install the plugin, then in Claude Code:

- "帮我看一下最近的企微文档"
- "读取这个表格 e3_xxx 的数据"
- "搜索标题包含'周报'的文档"
- "把 A1 单元格改成'已完成'"
- "我需要关注哪些任务？"

## License

MIT
