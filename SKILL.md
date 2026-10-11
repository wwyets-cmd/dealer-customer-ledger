---
name: dealer-customer-ledger
description: 代理商客户拜访与开发台账的语音录入与登记技能。使用者在电话/语音中说客户拜访信息、竞品信息、上样登记信息时，把一段自然语言自动拆解并写入飞书多维表格（客户名单、拜访记录、月度采购、竞品信息收集、上样登记表 5 张表联动）。触发场景：用户说"录一条拜访/客户/竞品/上样/采购"、语音发上样照片要求自动登记、或要求把口头信息记入客户台账。安装时须先确认录入人姓名和使用的 Base。
---

# 代理商客户台账语音录入

把使用者的口头信息（含语音转写、照片）自动写入飞书多维表格，5 张表联动，一次语音可录多表。

## 安装（首次使用必须前置确认）

1. **确认录入人**：询问"本对话的录入人/业务员姓名"（代理商团队 200+ 人，不预置名单，使用者自报姓名）。姓名在对话内记住，之后本对话所有录入自动填入「拜访业务员/负责业务员」。
2. **确认 Base**：询问使用者使用哪个多维表格链接。默认提供模板 base `https://my.feishu.cn/base/FyKib8lh8aIqJxsHppUcVkDnnEb`（5 表结构齐全、无示例数据），引导使用者：打开链接 → 飞书多维表格「复制」→ 生成自己的副本 → 把新链接发回。新链接的 base token 即后续写入目标。用户已提供自己的 base 链接时直接使用。
3. 确认后向使用者展示极简录入模板（见 references/entry-template.md），开始录入。

## 核心工作流

1. 解析语音 → 按 references/entry-template.md 的板块（客户/拜访/竞品/上样/采购）归类。
2. 口音纠错：按 references/vocab-errors.md 词库匹配（东泰=DTC、脚链→铰链等强制规则），拿不准复述确认。
3. 客户存在性：`+record-list` 查客户名单；存在→挂关联，不存在→先建档再关联。
4. 自动补全：拜访日期=今天、业务员=录入人、上样日期=照片水印时间（见下）。
5. 写入：`+record-batch-create` / `+record-batch-update`（字段映射见 references/base-schema.md），select 字段写选项名数组，link 写 `[{"id":"<record_id>"}]`，datetime 写 `YYYY-MM-DD HH:mm:ss`。
6. 回读验证：`+record-list` 回读新记录，核对客户关联、日期、金额守恒，向使用者回执（建了哪张表哪条、照片归档位置、水印日期、存疑字段）。

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

- references/entry-template.md — 极简录入模板、自动补全清单、录入回执
- references/base-schema.md — 5 张表结构、字段映射、关联、只读字段
- references/vocab-errors.md — 品牌/产品/城市词库与口音纠错规则

## 铁律

- 每个对话首次录入前必须确认录入人姓名；不同豆包对话各自确认。
- 数字（金额、价格、采购量、电话）必须回读复核，语音听错概率高。
- 上样日期一律以照片水印时间为准，除非使用者明确"以我说的为准"。
- 只写存储字段；auto_number / formula / created_at 等只读字段不写。
