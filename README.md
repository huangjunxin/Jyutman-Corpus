# Jyutman-Corpus

粤语文丛（Jyutman）的文本数据仓。

## 定位

本仓存放站点消费的 OCR 文本数据。数据来自上游管道 JyutmanDataPipeline 的发布层，
站点只读消费，绝不回写上游，也不消费上游 `work/` 层。

## 内容组织

| 目录 | 内容 |
|---|---|
| `data/` | 发布层 jsonl 快照（`articles` / `pages` / `issues`），随上游 release 更新 |
| `schemas/` | 字段语义说明，与上游数据契约对齐 |
| `rights/` | 版权逐件登记（底本年份、藏本、来源链接、版权判定） |
| `translations/` | 白话报今译（269 篇，AI 初译未校订）与三语料粤拼音表；说明见 [TRANSLATION_NOTES.md](translations/TRANSLATION_NOTES.md) |

`data/` 待上游首次发布后填充（当前仅有 `.gitkeep`）；`rights/` 已有 Phase 0 产出的首版登记（5 已核 / 4 待核），随核查进展更新。

## 许可

本仓文本数据采用 CC BY-SA 4.0 许可协议（见 [LICENSE](LICENSE)）。
底本为 1842 至 1907 年出版物的公有领域内容，扫描来源为 Internet Archive。
扫描影像本体不在本仓；影像默认不公开，须逐件核查版权后另行决定是否开放。
