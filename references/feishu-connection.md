# 飞书连接层适配（Feishu Connection · 平台无关）

> 本技能的业务规则、表结构、录入流程与所用平台无关；唯一随平台变化的是**"用什么方式读写飞书多维表格"**。本文件把这一层抽象出来：无论你在豆包工作、WorkBuddy、Codex 还是其它 Agent 平台，都只需在以下三种连接方式中选一种，并完成同样的一组操作。业务文档（SKILL.md / base-schema.md 等）里说的"通过飞书连接执行 X"，都对应本文件的操作。

## 一、三种连接方式（按平台选一种）

| 方式 | 适用平台 | 鉴权 | 说明 |
|---|---|---|---|
| **A. lark-cli** | 豆包工作（内置 lark-cli 的沙箱环境） | 命令行登录/用户授权（`--as user`） | 平台已封装好多维表格命令，最省事；具体命令见各平台安装文档 |
| **B. 飞书 MCP 连接器** | WorkBuddy（内置「飞书」连接器）、Codex / 任何支持 MCP 的 Agent | 平台连接器扫码授权，或填自建应用 App ID / App Secret | 以"工具"形式暴露：查表、查记录、新建/更新记录、上传附件 |
| **C. 飞书开放平台 HTTP API** | 任何能发 HTTP 请求的环境（Codex 脚本/自建后端兜底） | 自建应用 `app_id`+`app_secret` 换 `tenant_access_token`，或用户 OAuth token | 最通用，不依赖特定平台；需自建应用并开通多维表格权限 |

> 选型优先级：平台自带飞书连接器/MCP 时用 **B**；豆包沙箱用 **A**；都没有时用 **C**（HTTP API）。三种方式落到同一张表、同一套字段，业务规则完全一致。

## 二、连通性自检（首次配置必做）

能**列出该 Base 的数据表**即代表鉴权、权限、Base token 都正确。

- **A**：`lark-cli base +table-list --base-token <app_token>`
- **B**：调用连接器"列出数据表/查询表"工具，传入 Base 链接或 app_token。
- **C**：`GET https://open.feishu.cn/open-apis/bitable/v1/apps/{app_token}/tables`，Header 带 `Authorization: Bearer <tenant_access_token>`。

读不到表通常是：① 没开通多维表格权限；② 自建应用未被加为该文档协作者（见第五节）；③ Base 链接里的 token 取错。

## 三、必需操作 ↔ 三种实现对照

| 操作语义 | A. lark-cli | B. 飞书 MCP | C. 开放平台 HTTP API |
|---|---|---|---|
| 列数据表 | `base +table-list` | 查询表/数据表列表 | `GET /bitable/v1/apps/{app}/tables` |
| 列字段 | `base +field-list --table-id` | 查询字段 | `GET /bitable/v1/apps/{app}/tables/{table}/fields` |
| 列视图 | `base +view-list` | 查询视图 | `GET .../tables/{table}/views` |
| 查/搜记录（查重用） | `base +record-list` / `+record-search` | 查询记录 / 搜索记录 | `GET .../records`（支持 `filter`、分页 `page_token`） |
| 批量新建 | `base +record-batch-create` | 新建记录/批量新建 | `POST .../records/batch_create` |
| 更新记录 | `base +record-batch-update` | 更新记录 | `POST .../records/batch_update` |
| 删除记录（清空多余空记录） | `base +record-delete` | 删除记录 | `POST .../records/batch_delete` |
| 上传附件（上样照片） | `base +record-upload-attachment` | 上传附件/文件 | 先 `POST .../records/{record_id}/assets` 拿 `file_token`，再把 `[{file_token}]` 写回附件字段 |
| 建表/字段/视图/看板（结构对账） | `base +table-create / +field-create / +view-create / +dashboard-block-create` | 若连接器提供"建表/建字段"工具则用；否则提示人工 | 建表 `POST .../tables`、建字段 `POST .../fields`、建视图 `POST .../views`；看板组件能力以官方接口为准 |

> 所有 HTTP 路径前缀 `https://open.feishu.cn/open-apis`，`{app}` = Base app_token（多维表格链接 `https://…/base/<app_token>` 中 `/base/` 后那段）。

## 四、字段值写法（重点：三种连接的差异）

| 字段类型 | 取值写法 | 备注 |
|---|---|---|
| 文本 text / 主字段 | 字符串 | lark-cli 与 MCP 直接传字符串；HTTP API 文本可传字符串，富文本传对象数组 |
| 单选 select | 选项名字符串，如 `"轻高定"` | 选项不存在时，多数连接会自动新增；不自动新增则先补选项 |
| 多选 multiSelect | 字符串数组 `["A","B"]` | |
| 数字 number | number | 单价、数量、总金额；退货为负 |
| 日期 datetime | **A：`YYYY-MM-DD HH:mm:ss`；C：毫秒级时间戳（number）；B：按工具 schema（多为毫秒或 ISO）** | 同一份业务在不同连接下日期格式不同，务必按所用连接转换 |
| 勾选 checkbox | `true` / `false` | |
| 关联 link | 关联记录的 record_id 集合 | A 常用 `[{"id":"rec…"}]`；C 用 record_id 数组（以接口版本为准）；先查重拿到客户 record_id 再写 |
| 附件 attachment | `[{"file_token":"…"}]` | 先上传拿 file_token 再回填 |
| 公式 formula / 自动编号 / 创建时间 | **只读，不写** | 这些由系统计算，任何连接都不要写入 |

> 不确定某种连接的字段格式时：先建一条**测试记录**验证，再批量写入；禁止凭猜测批量灌数据。

## 五、权限与常见坑（HTTP API / MCP 自建应用）

1. **开通权限 scope**：自建应用在飞书开放平台「权限管理」至少开通多维表格读写（如 `bitable:app` 或记录的 create/read/update 等），并发布/安装到对应企业。
2. **把应用加为文档协作者**：自建应用即使有 scope，默认也访问不到具体某份表。需要在该多维表格「分享」里把**应用（机器人）添加为协作者**并给「可编辑」，否则报无权限。lark-cli/MCP 走用户本人授权时，则要求该用户本人对副本有「可编辑」权限。
3. **业务员权限**：按 permission-guide.md，业务员一律「可编辑」、老板/主管「可管理」；链接密码只拦浏览器陌生访客，不影响 API/MCP 写入（见 permission-guide）。
4. **限流**：建表/字段/视图/看板、批量写入时**串行、间隔 2–3 秒**，失败按错误码退避，不要并发猛打。
5. **写后必回读**：任何一种连接写完，都要重新查询该记录核对关联、日期、金额，再向使用者回执；发现多产生的空主字段记录立即删除。
6. **凭证安全**：`app_id`/`app_secret`/token 只放在环境变量或平台连接器的密钥配置里，**严禁写进任何技能文件、AGENTS.md 或提交进 Git 仓库**。

## 六、结构对账在各连接下的能力边界

- 结构对账（见 base-schema.md 末尾）要求建表、字段、视图、看板组件。lark-cli 支持最全；HTTP API 支持建表/字段/视图，**看板组件（dashboard）部分接口可能不开放或有限制**；部分 MCP 连接器只暴露记录读写、不暴露建字段。
- 当所用连接**不支持自动建某类结构**（典型：看板组件、公式字段）时：不要假装完成，在回执里列出"需人工在飞书界面补建的项"，并给出组件名、数据源表、字段和口径（base-schema.md 的 Changelog 已列全），让管理员几分钟内手动补好。
