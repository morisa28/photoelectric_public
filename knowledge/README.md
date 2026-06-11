# 光电效应论文知识库

本目录保存项目级论文知识库，来源为 `papers/` 下已下载并登记在 `papers/metadata.csv` 的 32 篇光电效应相关论文。知识库服务于光电效应实验模拟仿真平台的后续开发，重点支持原理动画区、3D 微观小窗、伏安特性曲线、截止电压讲解、实验报告和知识图谱。

## 当前状态

- 已索引论文：32 篇
- 核心论文：20 篇
- 支撑论文：12 篇
- 文件格式：PDF 为主，P030 为 CAJ，CAJ 暂只使用元数据和题名线索
- 生成脚本：`node scripts/papers/build-knowledge-base.mjs`

## 资料边界

- `papers/` 保存用户授权下载的论文原始文件。
- `knowledge/` 只保存元数据、审查结论、摘要卡片、主题索引和项目映射。
- 不在知识库中复制论文全文或长段原文。
- 不保存账号、Cookie、验证码、下载接口临时参数或其他登录凭证。

## 核心文件

- `paper-index.csv` / `paper-index.json`：论文基础索引。
- `review-rubric.md`：论文审查评分规则。
- `review-results.csv`：逐篇审查结果。
- `claims.json` / `claims.md`：结构化知识结论。
- `project-mapping.md`：知识结论到项目模块的映射。
- `topics/*.md`：按主题整理的知识页。
- `papers/P*.md`：单篇论文项目卡片。
- `source-coverage.json`：正文抽取和关键词覆盖情况。

## 使用方式

开发时优先读取：

1. `project-mapping.md`：确定模块应该参考哪些主题和论文。
2. `claims.md`：查看可直接转化为模型、文案或测试的结论。
3. `topics/*.md`：查看某一主题下的支撑论文。
4. `papers/P*.md`：复核单篇论文的项目用途。

只有当某个结论需要确认具体实验数据、图表或页码时，再打开对应的 `papers/*.pdf` 或 `papers/*.caj` 原文。
