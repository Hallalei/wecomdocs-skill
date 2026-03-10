---
name: wecomdocs
description: |
  操作企业微信文档（WeCom Docs / doc.weixin.qq.com）中的在线表格。当用户提及企业微信表格、WeCom 文档、doc.weixin.qq.com 链接、或出现 e3_/s3_ 前缀的文档 ID 时触发。适用场景：搜索和浏览文档、浏览共享空间、读取表格数据、写入单元格、查看编辑历史和版本对比、查看文档详情。即使用户没有明确说"企业微信"，只要上下文涉及 doc.weixin.qq.com 域名或相关文档操作都应触发。当用户需要从企微表格中查询信息（如任务、进度、周报数据）时也应触发。
---

# WeCom Docs 表格操作（wecomdocs-cli v0.6.x）

## 命令速查表

| 目标 | 命令 |
|------|------|
| 列出最近文档 | `wecomdocs-cli ls` |
| 浏览空间 | `ls --spaces` / `ls --space <id>` / `ls --space <id> --folder <fid>` |
| 搜索文档（仅标题） | `wecomdocs-cli search "关键词"` |
| 查看标签页 | `read <id> --tabs` |
| 读取表格 | `read <id> -s <scode>` |
| 筛选行（保留表头） | `read <id> -s <scode> --grep "关键词"` |
| 填充合并单元格 | `read <id> -s <scode> --fill-merged` |
| 指定行范围 | `read <id> -s <scode> --rows "5-20"` |
| 组合筛选 | `read <id> -s <scode> --fill-merged --grep "王雷"` |
| 快速读取大表 | `read <id> -s <scode> --fast` |
| 读取所有标签页 | `read <id> -s <scode> --all-tabs` |
| 写入单元格 | `write <id> --data "A1=值" -y` |
| 批量写入 | `write <id> --data "A1=值,B1=值" -y` / `write <id> --file changes.csv -y` |
| 预览写入 | `write <id> --data "A1=值" --dry-run` |
| 编辑历史 / 版本对比 | `history <id>` / `history <id> --diff 100..200` |
| 登录 / 退出 | `login` / `logout -y` |

常用选项：`--tab "名称"` / `--fast` / `--format json` / `-s <scode>` / `--line-numbers` — 完整：`<command> --help`

## 写入安全规则

<HARD-GATE>
**写入操作绝不可跳过确认流程。**

1. **展示写入计划**：列出每个单元格的当前值 → 新值
2. **耗时预告**：每个单元格约 1-2 秒，写入前告知用户预计耗时
3. **等待用户明确确认** — 未确认或拒绝时不执行
4. **写入后验证** — 用 `read` 或 `history --diff` 确认写入成功

写入操作始终在主线程执行，不委托给 Agent。
读取操作为只读，无需确认。
</HARD-GATE>

- write 命令必须加 `-y`（非 TTY 环境）。大批量先 `--dry-run` 预览。
- 写入验证：已有范围 → `read` | 新区域 → `history --diff`。
- 值含逗号或大量单元格 → 用 `--file` 替代 `--data`。

## 查询指引

- **直接操作**（读表格、写单元格、看标签页）直接执行，不需要额外流程
- **分析型查询**（"我该关注什么"、"进度怎样"）流程：
  1. `ls` 定位相关文档
  2. 确认用户**姓名 + 角色**（参见"用户身份识别"）
  3. 按角色需求读取数据（`--fill-merged --grep "姓名"` 组合）
  4. 以角色专属模板输出结论
- 多文档查询时，将独立的 read 命令放在同一消息的多个 Bash 调用中并行执行
- 合并单元格场景加 `--fill-merged`，CLI 自动向下填充空值
- **复杂合并表格**（OKR、多层嵌套）解析策略：
  1. 首选 `--fill-merged --grep "姓名"` 组合精准提取
  2. 列对齐混乱时改用 `--format csv` 或 `--format tsv`
  3. 先 `--rows "1-3"` 单独读表头理解列结构，再读数据行
  4. 大型 OKR 表：先缩小范围（grep + rows），避免全量读取后手动解析
- 优先用 `--grep` 筛选、`--rows` 截取，而非全量读取后手动分析
- `--rows` 行号：row 1 = 第一个数据行（表头自动保留，不计入范围）；处理管道顺序：`--fill-merged` → `--grep` → `--rows`（--rows 作用于 grep 后的结果）
- 先 `--tabs` 了解结构，再按需读取；大表（>50 行）用 `--fast`；`--fast` 可与 `--fill-merged` / `--grep` 自由组合
- `search` 仅搜标题；搜内容用 `read | grep` 或 `--grep`；无结果则 `ls` 浏览或问用户

---

## 输出原则

分析型查询（非直接读取）的输出必须遵循：

1. **最多 3-5 项，强制排优先级** — 不做信息平铺，用户需要更多可以追问
2. **行动优先** — 每项以动词开头（补填、同步、确认、推进），不写纯状态描述
3. **附数据依据** — 每项引用具体来源（文档名 + 行号或内容），不说空话
4. **只写用户相关** — 排除用户未参与的团队动态
5. **末尾一行注明数据来源** — `> 数据来源：文档A、文档B`

### 角色专属输出模板

根据用户角色调整输出重点：

**PM**：每项包含 → 需求/项目状态 | 对接人与对接事项 | 待决策项 | 时间节点与风险
**工程师**：每项包含 → 开发任务与进度 | 技术阻塞 | 上下游依赖 | 本周交付物
**TL / 负责人**：每项包含 → 团队整体进展 | 资源瓶颈 | 需升级/拉齐的事项

### 对比示例

❌ 坏：`🔴 OKR 进度更新缺失：你负责的 3 项知识库任务全部没有填写周进度...` → 纯状态陈述，无行动指引
❌ 坏：`📌 团队整体动态` → 平铺不相关信息，稀释重点

✅ 好：
```
## 本周你需要关注的 3 件事
1. **补填 OKR 周进度** — 你的 3 项知识库 KR 从未更新过进度，本周 review 前需要补上（OKR 表第 20-22 行）
2. **和亚光同步 RAG 优化进展** — "RAG 召回效果优化"已在开发中（里程碑表第 27 行），你的 rewrite/rerank KR 与此直接相关
3. **确认 2 项 KR 的 H1/H2 归属** — "分级召回"和"动态库"标注"可能 H2"（OKR 表第 23-24 行）
> 数据来源：平台产品 2026 H1 OKR、平台 AI 里程碑-项目
```

## 用户身份识别

身份信息持久化存储在 `~/.wecomdocs-cli/user_profile.json`，格式：
```json
{
  "name": "真名（用于 grep）",
  "displayName": "完整显示名",
  "role": "PM | 工程师 | TL",
  "updatedAt": "ISO 8601 时间戳"
}
```

### 识别流程

1. **读取已保存身份**：用 Read 工具读取 `~/.wecomdocs-cli/user_profile.json`
   - 文件存在且包含 `name` + `role` → 直接使用，输出一行确认：`已识别身份：{name}（{role}）`
   - 文件不存在或字段缺失 → 进入首次设置

2. **首次设置**（仅在无已保存身份时执行）：
   - `wecomdocs-cli ls` → 从"修改人"提取括号内真名（`Ranchoddas（王雷）` → `王雷`）
   - 一次提问合并确认：`从文档记录看你是"王雷"，对吗？你的角色是？（PM / 工程师 / TL）`
   - 用户确认后，用 Write 工具将身份写入 `~/.wecomdocs-cli/user_profile.json`

3. **身份更新**：用户说 "我不是XX" / "我是YY" / "角色改成XX" / "更新身份" 时：
   - 按用户提供的信息更新对应字段
   - 重新写入 `~/.wecomdocs-cli/user_profile.json`
   - 确认：`身份已更新：{name}（{role}）`

- 用**真名** grep，角色影响输出视角（见"输出原则 → 角色专属模板"）

## 文档浏览与搜索

- 空间导航：`ls --spaces` → `ls --space <id>` → `ls --space <id> --folder <fid>` → `doc-info <id>`
- `search` 仅搜标题。无结果时：`ls` 浏览 → 空间导航 → `read --grep` → 问用户

## 输入格式 + 环境检查

- 文档 ID：`e3_xxx` | 完整 URL（自动提取 ID+scode）| **scode** 访问他人文档时需要
- 环境：`wecomdocs-cli --version`（未安装 → `npm i -g wecomdocs-cli`）、`~/.wecomdocs-cli/session.json`（不存在 → `login`）、Playwright 报错 → `npx playwright install chromium`

## 错误处理

| 错误 | 解决 |
|------|------|
| 未登录 / session 过期 / 401/403 | `wecomdocs-cli login` 重新登录 |
| session 异常 | `wecomdocs-cli logout -y` 后重新 `login` |
| 需要 playwright | `npx playwright install chromium` |
| 标签页名称错误 | 先 `read <id> --tabs` 确认 |
| 非交互模式写入报错 | write 命令必须加 `-y` |
| `--folder` 无效 | 必须配合 `--space` 使用 |
| 文档无权限 | 确认 scode 是否正确 |
| search 无结果 | search 仅搜标题，改用 `ls` 浏览或 `--grep` 搜内容 |
