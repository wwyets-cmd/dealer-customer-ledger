# Codex 安装与连接（Codex CLI / 兼容 AGENTS.md 的 Agent）

适用：**Codex CLI** 或任何在项目目录读取 **`AGENTS.md`** 的 Agent。Codex 的飞书连接优先用 **飞书 MCP**（方式 B），未配置时用 **开放平台 HTTP API**（方式 C）。

## 第 1 步：放置技能

1. 把 `dealer-customer-ledger-universal` 作为项目根目录（或放入你的项目里），确保 **`AGENTS.md` 在其中**。
2. Codex 启动时会读 `AGENTS.md`，并按其指示读取 `SKILL.md` 与 `references/`、`platforms/`。
   - 若希望所有项目都可用，可把本文件夹放到 `~/.codex/` 下并用全局 `AGENTS.md` 引用；单项目用则放项目目录即可。

## 第 2 步：连接飞书（二选一）

### 方式一：飞书 MCP（推荐）

编辑 `~/.codex/config.toml`，新增一个飞书 MCP server（`command`/`args` 或 `url` 以你选用的**飞书 MCP server 官方说明**为准，下方为结构示例）：

```toml
[mcp_servers.feishu]
command = "npx"
args = ["-y", "<你的飞书 MCP server 启动命令或包名>"]
# 远程 MCP 时改为： url = "https://<your-mcp-endpoint>"
env = { FEISHU_APP_ID = "cli_xxx", FEISHU_APP_SECRET = "xxx" }
```

重启 Codex，确认其工具列表里出现飞书多维表格相关工具（查表/查记录/新建/更新/上传附件）。

### 方式二：飞书开放平台 HTTP API（兜底）

1. 飞书开放平台创建**自建应用**，在「权限管理」开通多维表格读写权限（`bitable:app` 及记录 create/read/update），发布并安装到企业。
2. 在目标多维表格「分享」里把**应用（机器人）加为协作者、给「可编辑」**。
3. 用环境变量提供凭证（不要写进文件）：
   ```bash
   export FEISHU_APP_ID="cli_xxx"
   export FEISHU_APP_SECRET="xxxx"
   export FEISHU_BASE_TOKEN="xxxxxxxx"   # 副本链接 /base/ 后那段
   ```
4. 换取访问令牌：
   ```http
   POST https://open.feishu.cn/open-apis/auth/v3/tenant_access_token/internal
   Content-Type: application/json
   { "app_id": "cli_xxx", "app_secret": "xxxx" }
   ```
5. 后续调用都带 `Authorization: Bearer <tenant_access_token>`，多维表格路径前缀：
   `https://open.feishu.cn/open-apis/bitable/v1/apps/{app_token}/...`
   （端点、字段值格式见 `references/feishu-connection.md`）。

## 第 3 步：连通自检

让 Codex **列出该 Base 的数据表**（MCP 的"列出数据表"工具，或 `GET .../tables`）。能返回 5 张表即鉴权、权限、token 全部正确。

## 第 4 步：首次对话

1. 告知**录入人正式全名**（同一人跨对话必须同名）。
2. 按 `references/base-copy-guide.md` 复制模板、提供**副本链接**。
3. Codex 会：连通自检 → 按 `base-schema.md` 结构对账 → 建个人视图 → 展示录入模板，然后开始录入。

## 常见问题

- **报无权限 / 1254030**：自建应用未加为副本协作者，或权限未开通/未发布。
- **日期写不进或报错**：HTTP API 的日期字段是**毫秒时间戳**（不是 `YYYY-MM-DD`）；MCP 按其工具 schema。
- **附件传不上去**：先 `POST .../records/{record_id}/assets` 拿 `file_token`，再把 `[{ "file_token": "..." }]` 回填附件字段。
- **关联字段写不进**：先查客户记录拿到 record_id，link 传记录 id 集合，格式以接口版本为准。
- **Codex 没读规则**：确认 `AGENTS.md` 在项目根且其中列出的 `SKILL.md`、`references/` 文件路径无误。
