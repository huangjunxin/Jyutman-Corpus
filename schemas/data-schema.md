# 数据字段说明

本文件精简誊写自站点仓 `docs/03-data-contract.md`，说明发布层三张表的字段语义与已知坑。
两者如有出入，以站点仓文档与上游实现为准。上游 schema 可能演进，消费方须对未知字段容错。

## 1. 消费边界

- 唯一消费点：上游发布层 `data/<corpus>/{articles,pages,issues}.jsonl`。
- 不消费上游 `work/` 层（内部工作产物，格式不稳定、随时可能变）。
- 不回写上游；编码 UTF-8，jsonl 每行一个 JSON 对象（`ensure_ascii=False`）。
- 时效：上游发布 release 后站点才同步，不追踪未发布的前沿状态。

## 2. 数据模型

```text
corpus（语料 slug，如 gd-vernacular-paper）
  └─ issue（期号）
       └─ page（叶，扫描序号）
            └─ article（文章，检索与展示的基本单位）
```

注意：corpus slug（如 `gd-vernacular-paper`）与文章 ID 前缀（如 `gdvp`）是两套标识，勿混用。

## 3. articles.jsonl

字段（值为占位示意，非真实数据）：

```jsonl
{"id":"<前缀>-<期>-<页>-<序>","corpus":"...","issue":"...","issue_date":"...","page":"...","seq":0,"source_image":"...","title":null,"text":"...","unclear":[],"status":"draft"}
```

| 字段 | 类型 | 说明 |
|---|---|---|
| `id` | string | `<前缀>-<期>-<页>-<序>`（示例 `gdvp-01-009-01`）；新语料前缀会退化为整条 slug，应视为不透明键 |
| `corpus` | string | 语料代号 |
| `issue` | string | 期号（在语料内唯一） |
| `issue_date` | string \| null | 依赖上游 ISSUE_INFO 表，新语料可能为 null，排序与展示必须容忍 |
| `page` | number | 扫描叶序号（整数），是 URL 与影像定位主键，不是原书印刷叶码 |
| `seq` | number | 该叶内的文章序号 |
| `source_image` | string | 报纸为单页 JPG；书本为整本 PDF（没有单页影像） |
| `title` | string \| null | 无题文章为 null，展示时需生成显示名 |
| `text` | string | 正文，`\n\n` 分段；汉字、传教士罗马字、英文三语混排在同一字段 |
| `unclear` | array | 目前恒为空数组，不要为它设计 UI |
| `status` | string | `draft` / `verified` / `needs_review`，由页级状态传播而来 |

## 4. pages.jsonl

整页全文、该叶的文章数与 `status`。字段（已实测）：`corpus` / `issue` / `issue_date` /
`page`（int）/ `source_filename` / `source_image` / `text` / `unclear` / `articles`（int）/ `status`。
上游 schema 可能演进，同步脚本仍需对未知字段容错。

## 5. issues.jsonl

期号元数据与校对进度统计。字段（已实测）：`corpus` / `issue` / `issue_date` /
`date_note` / `pages`（int）/ `articles`（int）/ `verified_pages`（int）/
`needs_review_pages`（int）/ `workspace`（上游工作区路径，仅内部参考，不对外展示）。

`date_note` 是期号日期的考证说明，属研究性内容，应展示而非丢弃。

## 6. 已知坑

| # | 坑 | 应对 |
|---|---|---|
| 1 | 新语料 ID 前缀退化为整条 slug | ID 当不透明键，解析失败退化为整体 slug |
| 2 | 正文含未识别字 `□`；`status` 有三态 | UI 明示校对状态；`□` 用独立样式，不静默丢弃 |
| 3 | `unclear` 恒为空，疑点实际在上游 review_checklist | 不为它设计 UI，待上游结构化后再接 |
| 4 | 三语混排在同一个 `text` 字段 | 自建分段策略；检索侧另建罗马字归一字段 |
| 5 | 正文含 `［插圖］` / `［空白頁］` / `［現代襯頁］` 标记 | 渲染时特判为占位块或说明文字 |
| 6 | 报纸扫描仅约 968×1452px | 阅读器限制最大放大倍率并在 UI 明示 |
| 7 | `source_image` 含中文文件名 | 上传 R2 时改写为 ASCII object key，manifest 保留映射 |
| 8 | `genre` / `column` / `series` / `author` 为上游预留字段 | schema 用 passthrough 容错，UI「有则显示」 |
