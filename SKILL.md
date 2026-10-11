---
name: dealer-customer-ledger
description: 代理商客户拜访与开发台账的语音录入与登记技能。使用者在电话/语音中说客户拜访信息、竞品信息、上样登记信息、回单/订单信息，或每月直接发订单明细数据时，把一段自然语言/表格自动拆解并写入飞书多维表格（客户名单、拜访记录、回单记录、竞品信息收集、上样登记表 5 张表联动）。触发场景：用户说"录一条拜访/客户/竞品/上样/回单"、语音发上样照片要求自动登记、每月发订单明细要求导入识别、或要求把口头信息记入客户台账。安装时须先确认录入人姓名和使用的 Base。
---

# 代理商客户台账语音录入

把使用者的口头信息（含语音转写、照片）自动写入飞书多维表格，5 张表联动，一次语音可录多表。

## 安装（首次使用必须前置确认）

1. **确认录入人**：询问"本对话的录入人/业务员姓名"（不预置名单，使用者自报姓名）。姓名在对话内记住，之后本对话所有录入自动填入「拜访业务员/负责业务员」。
2. **确认 Base（代理商自行复制模板）**：每位代理商**复制一份模板**得到自己的表，录进自己副本。模板链接 `https://my.feishu.cn/base/FyKib8lh8aIqJxsHppUcVkDnnEb`（5 表结构齐全、无示例数据）。**完整复制教程见 references/base-copy-guide.md**（打开模板 → 登录飞书 → 「⋯」→ 复制 → 命名 → 确认 5 表 → 把副本链接发回）。注意：模板需管理员对代理商开「可编辑」权限才能复制；不要在模板母版上直接录数据。
3. 确认后向使用者展示详细录入模板（见 references/entry-template.md，含口语示例），开始录入。

## 核心工作流

1. 解析语音/表格 → 按 references/entry-template.md 的板块（客户/拜访/竞品/回单/上样）归类；使用者信息说得越详细越好。
2. 词库匹配（内部处理，不向使用者展示纠错清单）：按 references/vocab-errors.md 归一（东泰=DTC、脚链→铰链、智能收纳舱→升降机），拿不准复述确认。
3. 客户存在性：`+record-list` 查客户名单；存在→挂关联，不存在→先建档再关联。
4. 自动补全：拜访日期=今天、业务员=录入人、上样日期=照片水印时间（见下）。
5. 写入：`+record-batch-create` / `+record-batch-update`（字段映射见 references/base-schema.md，**select 单选字段写选项名字符串、多选写数组**，link 写 `[{"id":"<record_id>"}]`，datetime 写 `YYYY-MM-DD HH:mm:ss`）。
6. 回读验证：`+record-list` 回读新记录，核对客户关联、日期、金额守恒，向使用者回执（建了哪张表哪条、照片归档位置、水印日期、存疑字段）。

## 每周订单数据导入（每周一次）

使用者每周把销售/订单明细 Excel（.xls/.xlsx/CSV/截图）发来，代理自动分析识别后写入**回单记录**。导出信息须包含：订单日期、客户名、产品、数量、单价、总金额。**完整导入教程见 references/import-guide.md**（字段要求/列名别名/产品归一/客户匹配/数字复核/导入后校验），按该文档执行。

## 上样照片登记（特殊流程）

- 使用者发照片+说公司名：先读照片，识别**水印相机时间**（年月日时分），写入「上样日期」（不默认今天；水印看不清先问）。
- 照片归档：门头签单合照/协议合照/上样前照片/上样后照片四个附件字段，按使用者说明分类；没说则由内容判断，拿不准问。附件用 `+record-upload-attachment --field-id <字段id> --file <path>` 上传。
- 老客户自动挂「关联客户」；新客户先建档。

## 常用命令（配合 lark-base skill 使用）

- 查表/字段：`lark-cli base +table-list` / `+field-list --table-id`
- 查客户：`+record-list --table-id <客户名单id>`
- 写记录：`+record-batch-create`（create_records 为字段 map 数组）
- 更新：`+record-batch-update`（update_records 为 {record_id:{字段:值}}）
- 附件：`+record-upload-attachment`
- 验证：写后必须 `+record-list` 回读，禁止仅凭写成功宣称完成

## 参考文档

- references/entry-template.md — 详细录入模板（含口语示例）、自动补全清单、录入回执
- references/import-guide.md — 每周 Excel 订单数据导入教程（字段要求、识别规则、导入后校验）
- references/base-schema.md — 5 张表结构、字段映射、关联、只读字段
- references/vocab-errors.md — 品牌/产品/城市词库与口音归一规则
- references/base-copy-guide.md — 代理商复制模板的完整教程（打开→登录→复制→检查→发链接）

## 铁律

- 每个对话首次录入前必须确认录入人姓名；不同豆包对话各自确认。
- 数字（金额、价格、采购量、电话）必须回读复核，语音听错概率高。
- 上样日期一律以照片水印时间为准，除非使用者明确"以我说的为准"。
- 只写存储字段；auto_number / formula / created_at 等只读字段不写。
