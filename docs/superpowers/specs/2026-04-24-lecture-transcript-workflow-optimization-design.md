# Lecture-Transcript Skill Workflow Optimization Design

## Overview

优化lecture-transcript skill的处理流程，实现章节并行处理能力，提升大文档处理效率。

---

## Workflow Summary

```
用户执行 /lecture-transcript → 提供PPT内容
    ↓
主session解析文档 → 检测章节结构 → 显示章节列表
    ↓
用户确认/调整章节划分
    ↓
为每个章节创建Task
    ↓
显示执行信息 → 询问是否并行执行
    ↓
[并行] Spawn多个Agent处理各章节Task
[顺序] 主session逐个处理Task
    ↓
章节Agent：生成逐字稿 + 自检风格 + 保存文件
    ↓
失败处理：重试2次 → 仍失败则标记 → 合并时提示用户重试
    ↓
所有章节完成 → Spawn合并Agent
    ↓
合并Agent：拼接 + 风格一致性检查 + 内容完整性检查 → 输出最终文档
```

---

## Chapter Detection & Display

### Detection Logic

1. **页面标记识别**：查找PPT内容中的页面标记（如 "p1"、"p12"、"第1页"等格式）
2. **标题层级识别**：识别章节标题（如 "第一章"、"一、"、"1.1"等格式）
3. **内容逻辑分组**：根据内容主题变化判断章节边界

### Display Format

```
检测到 5 个章节：

| 章节 | 页码范围 | 标题 | 内容概要 |
|------|----------|------|----------|
| 1    | p1-p5    | 排序概述 | 排序定义、分类、基本概念 |
| 2    | p6-p12   | 插入排序 | 直接插入、折半插入、希尔排序 |
| ...  | ...      | ...  | ...      |
```

### User Adjustment Options

- 合并章节
- 拆分章节
- 调整章节边界页码

---

## Parallel Execution Decision

### Information Display

```
===========================================
执行方式选择
===========================================
章节数量：N 个
预计耗时：
  - 并行执行：约 X 分钟
  - 顺序执行：约 Y 分钟

资源提示：
  - 并行执行会同时占用 N 个Agent，资源消耗较高
  - 顺序执行资源消耗低，但耗时较长
===========================================

是否开启并行执行？
A) 是，并行处理所有章节
B) 否，按顺序逐个处理
C) 部分并行（指定哪些章节并行）
```

---

## Chapter Agent Responsibilities

每个章节Agent执行：

1. **读取章节内容**：从主session获取该章节的PPT原文
2. **生成逐字稿**：按skill风格要求转换为口语化讲解
3. **自检风格**：
   - 检查是否有"同学们"、"大家"等互动用语
   - 检查是否过于书面化（"其"、"即"、"所谓"）
   - 检查是否有PPT定义直接复制
4. **保存文件**：写入 `逐字稿_第{N}章.md`

### Agent Prompt Template

```markdown
你是大学讲课逐字稿生成Agent。

任务：处理第{章节号}章节
章节标题：{标题}
章节内容：{PPT原文}

要求：
1. 按照lecture-transcript skill的风格生成口语化逐字稿
2. 自检生成的内容是否符合风格要求
3. 保存结果到文件：逐字稿_第{章节号}章.md

风格要点：
- 使用"同学们"、"大家"等互动称呼
- 用生活例子引入抽象概念
- 避免PPT定义直接复制
- 避免书面用语（其、即、所谓）
```

---

## Failure Handling

### Retry Strategy

```
尝试生成逐字稿
    ↓
[成功] → 自检 → 保存 → Task标记completed
    ↓
[失败] → 重试（最多2次）
    ↓
    → 重试成功 → 继续流程
    → 重试失败 → Task标记failed，记录失败原因
```

### Status File

失败章节信息保存到 `处理状态.json`：

```json
{
  "chapters": [
    { "id": 1, "status": "completed", "file": "逐字稿_第1章.md" },
    { "id": 2, "status": "failed", "reason": "内容解析超时", "retry_count": 2 },
    ...
  ]
}
```

### Merge Failure Prompt

合并时检测到failed章节：

```
===========================================
⚠️ 章节处理失败提示
===========================================
第N章节处理失败，已重试2次。
失败原因：{reason}

处理选项：
A) 重试该章节
B) 跳过该章节，合并其他章节
C) 手动处理该章节
===========================================
```

用户选择重试后：
- Task状态重置为pending
- Spawn新Agent处理该Task
- 成功后继续合并流程

---

## Merge Agent Responsibilities

### Execution Flow

```
读取所有章节文件
    ↓
按章节顺序拼接
    ↓
风格一致性检查：
    - 称呼用语统一
    - 语气一致
    - 发现不一致 → 标记 → 修正
    ↓
内容完整性检查：
    - 章节衔接流畅
    - 无重复内容
    - 无概念遗漏
    - 发现问题 → 标记 → 补充衔接
    ↓
添加整体结构：标题、目录、开篇、结束语
    ↓
输出最终文档：逐字稿_完整版.md
    ↓
生成检查报告：合并检查报告.md
```

### Output Format

```markdown
# {课程标题} 逐字稿

## 目录
1. {章节1标题}
2. {章节2标题}
...

---

## 第一章 {标题}
{内容}

---

## 第二章 {标题}
{内容}

...

---

好了，今天我们讲完了{主题}。大家回去好好复习，下节课我们继续。
```

### Check Report

```markdown
# 合并检查报告

## 风格一致性
- ✓ 称呼用语统一
- ⚠ 语气差异：第X章节偏正式，已修正
- ✓ 讲解风格一致

## 内容完整性
- ✓ 章节衔接流畅
- ✓ 无重复内容
- ✓ 无概念遗漏

## 修正记录
- 第X章第Y行：将"A"改为"B"
```

---

## SKILL.md Updates

### Replace Current Workflow Section

将"处理流程（防止超时）"章节替换为：

```markdown
## 处理流程（并行优化版）

### 1. 章节解析阶段
用户执行 `/lecture-transcript` 并提供PPT内容后：
- 自动检测章节结构（页面标记 + 标题层级 + 内容逻辑）
- 显示章节列表（章节号、页码范围、标题、内容概要）
- 用户确认或调整章节划分

### 2. 任务创建阶段
- 为每个章节创建Task（使用TaskCreate工具）
- 显示执行选项：章节数、预计耗时、资源提示
- 询问用户是否并行执行

### 3. 执行阶段
**并行模式**：
- Spawn多个Agent，每个Agent处理一个章节Task
- 每个Agent独立完成：生成 + 自检 + 保存

**顺序模式**：
- 主session逐个处理Task
- 每完成一个章节，更新Task状态

### 4. 失败处理
- 单章节失败：自动重试最多2次
- 超过重试次数：标记failed，在合并阶段提示用户
- 用户可选择：重试、跳过、手动处理

### 5. 合并阶段
- 所有章节完成后Spawn合并Agent
- 合并Agent职责：拼接 + 风格一致性检查 + 内容完整性检查 + 输出最终文档
- 生成合并检查报告
```

### Keep Unchanged

- 目标
- 风格要求
- 写作指南
- 结构要求
- 禁止行为
- 使用方式
- 示例

---

## Implementation Notes

- Use TaskCreate/TaskUpdate/TaskList tools for task management
- Use Agent tool to spawn chapter processing agents and merge agent
- Use `isolation: worktree` for parallel agents if needed
- Status file `处理状态.json` tracks each chapter's processing state
- Output files: `逐字稿_第{N}章.md` (per chapter), `逐字稿_完整版.md` (final), `合并检查报告.md` (report)