# AGENTS.md — 代理商客户台账（Codex / 通用 Agent 入口）

本目录是「代理商客户拜访与开发台账」技能（dealer-customer-ledger），把使用者的语音/文字拜访信息、上样照片、每周订单明细自动写入飞书多维表格（5 表联动）。

## 开始任何任务前，必须完整阅读并严格遵守

- **`./SKILL.md`**：完整业务规则与工作流（文件顶部的 YAML frontmatter 是给豆包/WorkBuddy 等技能系统用的，Codex 可忽略；其下正文即本任务指令，必须照做）。
- **`./references/` 下全部文档**，至少先读：
  - `feishu-connection.md` —— 飞书连接层（三种连接、操作对照、字段值写法、权限、HTTP 端点）
  - `entry-template.md` —— 录入模板、必填规则、口语示例、工作群汇报文案
  - `base-schema.md` —— 5 张表结构/字段映射/只读字段/结构对账
  - `import-guide.md` —— 每周订单 Excel 导入
  - `vocab-errors.md`、`product-catalog.md` —— 口音归一与产品词库
  - `permission-guide.md`、`base-copy-guide.md`
- 安装与本平台连接的图形步骤见 `platforms/codex.md`。

> 业务规则以 SKILL.md / references 为**单一事实来源**，不得自创字段、表或流程；拿不准的字段值先问使用者，不猜。

## 本平台（Codex）如何连接飞书

1. **优先 MCP**：在 `~/.codex/config.toml` 配置飞书 MCP server（按 `platforms/codex.md`），之后用其暴露的多维表格工具读写（查表/查记录/新建/更新/上传附件）。
2. **兜底 HTTP API**：未配置 MCP 时，用飞书开放平台 HTTP API（自建应用，`app_id`/`app_secret` 换 `tenant_access_token`），端点与字段格式见 `references/feishu-connection.md`。
3. **凭证安全**：只从环境变量读取（如 `FEISHU_APP_ID`、`FEISHU_APP_SECRET`、`FEISHU_BASE_TOKEN`），**禁止硬编码、打印或写入任何文件/仓库**。
4. 首次连接先"列出数据表"做自检；自建应用需被加为目标 Base 的协作者（可编辑），否则无权限。

## 每个新对话的前两步

1. 询问并记住**录入人正式全名**（同一人跨对话必须同名）。
2. 让使用者按 `base-copy-guide.md` 复制模板、提供其**副本 Base 链接**；完成连通自检 → 按 `base-schema.md` 结构对账 → 建个人视图 → 展示录入模板后再开始录入。

之后的查重（一人一档）、必填校验、开发阶段自动流转、跟进 SLA、上样水印、每周导入、工作群汇报等，全部按 `SKILL.md` 执行。
