# 代理商客户拜访与开发台账（dealer-customer-ledger · 通用版）

语音/文字录入客户拜访信息，自动拆解并写入飞书多维表格，**5 表联动 + 数据看板**。一份技能包，**豆包工作、WorkBuddy、Codex 等任意 Agent 平台通用**。

## 表与看板

- **客户名单**（一人一档，统计主表；同客户只补全不重建）
- **拜访记录**（每次拜访一条）
- **回单记录**（每笔订单，不只首单；退货记负数）
- **竞品信息收集**（品牌/型号/价格/采购量；只记录不比价）
- **上样登记表**（水印相机照片，日期按水印时间；门头照+上样后照必传）
- **信息汇总看板**（拜访量、新开发、业绩/拜访排名、产品金额排名、竞品分析、今日/逾期跟进、上样超期预警）

## 为什么能"一份包通用"

- **业务规则只有一套**（查重、必填、开发阶段自动流转、跟进 SLA、口音纠错、工作群汇报、每周导入），平台无关，写在 `SKILL.md` + `references/`。
- **飞书连接层可插拔**：`references/feishu-connection.md` 统一了三种连接方式，各平台任选其一：

| 平台 | 入口文件 | 飞书连接 | 安装文档 |
|---|---|---|---|
| **豆包工作 / Doubao** | `SKILL.md` | lark-cli（内置） | `platforms/豆包工作.md` |
| **WorkBuddy / OpenClaw** | `SKILL.md` | 飞书 MCP 连接器 | `platforms/workbuddy.md` |
| **Codex / 兼容 AGENTS.md 的 Agent** | `AGENTS.md` → 引用 `SKILL.md` | 飞书 MCP（优先）或开放平台 HTTP API | `platforms/codex.md` |

## 快速开始

1. 按你的平台把本文件夹放到技能目录（见对应 `platforms/*.md`）。
2. 复制飞书模板，得到自己的副本（教程 `references/base-copy-guide.md`）：
   **https://my.feishu.cn/base/FyKib8lh8aIqJxsHppUcVkDnnEb**
3. 复制后按 `references/permission-guide.md` 配置协作者：业务员「可编辑」、老板/主管「可管理」。
4. 在新对话里告诉助手：**录入人正式全名** + **副本链接**。助手连通自检 → 结构对账 → 建个人视图，即可开始语音录入。

## 口语录入示例

> 录一下今天拜访的：济南华鑫全屋定制，老客户，轻高定，老板姓王，门店在历城工业北路88号，电话18888888888。今天对接升降机SL-A上样，客户有意向，但担心承重、且xx品牌HIG-560只报1200，我们没价格优势。下次10月25日前上门，带样板演示承重、出照明报价。竞品一条：xx HIG-560，1200，上月采购了5套。

录入核对后会自动生成一段可直接发工作群的汇报文案。

## 每周订单导入

每周把销售明细（Excel/CSV/截图，含订单日期、客户名、产品、数量、单价、总金额）发来，自动识别写入回单记录，见 `references/import-guide.md`。

## 目录结构

```
dealer-customer-ledger-universal/
├── README.md                 # 本文件（总览+快速开始）
├── SKILL.md                  # 技能主指令（平台无关，豆包/WorkBuddy 直接加载）
├── AGENTS.md                 # Codex 等 AGENTS 入口（引用 SKILL.md）
├── references/
│   ├── feishu-connection.md  # 飞书连接层（三种连接/操作对照/字段值写法/权限）
│   ├── entry-template.md     # 录入模板、必填规则、口语示例、工作群汇报
│   ├── base-schema.md        # 5 表结构、字段映射、只读字段、结构对账
│   ├── import-guide.md       # 每周订单导入教程
│   ├── vocab-errors.md       # 品牌/产品/城市词库与口音归一
│   ├── product-catalog.md    # 产品词库（276 型号系列）
│   ├── permission-guide.md   # 权限角色与防误删
│   └── base-copy-guide.md    # 模板复制教程
├── platforms/
│   ├── 豆包工作.md
│   ├── workbuddy.md
│   └── codex.md
└── price-reference/
    └── 炬森2026产品价格表-产品名称识别清单.xlsx
```

## 安全须知

- 飞书 `app_id` / `app_secret` / token 只放环境变量或平台连接器密钥，**不写进任何技能文件、不提交仓库**。
- 模板母版保持空、不在上面录数据；各代理商只在自己的副本里操作。
- 表格可设链接密码，密码只拦浏览器陌生访客，不影响 MCP/API 写入（见 `permission-guide.md`）。
