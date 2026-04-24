# Lecture-Transcript Workflow Optimization Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 更新lecture-transcript skill的处理流程，实现章节并行处理能力。

**Architecture:** 修改SKILL.md文件，将现有的"处理流程（防止超时）"章节替换为新的"处理流程（并行优化版）"章节，保留其他内容不变。

**Tech Stack:** Markdown skill文件，Claude Code内置工具（TaskCreate、Agent、AskUserQuestion）

---

## File Structure

| 文件 | 操作 | 职责 |
|------|------|------|
| `skills/lecture-transcript/SKILL.md` | 修改 | 替换第98-129行的处理流程章节 |

---

### Task 1: 更新SKILL.md处理流程章节

**Files:**
- Modify: `skills/lecture-transcript/SKILL.md:98-129`

**当前内容（需要替换的部分）：**
```markdown
## 处理流程（防止超时）

当PPT内容较多时，直接一次性生成会超时失败。必须按以下流程处理：

### 1. 先识别章节结构
首先分析PPT内容，识别出所有章节（按PPT的章节划分或内容逻辑划分），列出章节清单。

### 2. 按章节逐个生成
**必须一个章节一个章节地处理，不能一次性处理所有章节！**

流程：
```
第一轮：处理第1章节 → 生成文档（如：逐字稿_第1章.md）
第二轮：处理第2章节 → 生成文档（如：逐字稿_第2章.md）
第三轮：处理第3章节 → 生成文档（如：逐字稿_第3章.md）
...
最后一轮：合并所有章节文档 → 最终文档（如：逐字稿_完整版.md）
```

### 3. 每轮处理规则
- 每轮只处理**一个章节**
- 处理完一个章节后，等待用户确认继续下一章节
- 或使用"继续处理下一章节"指令自动推进
- 每个章节单独保存为一个文档文件

### 4. 最终合并
所有章节完成后，将所有章节文档合并为一个完整文档：
- 按章节顺序拼接
- 保持章节标题格式一致
- 添加总标题和目录（可选）
```

- [ ] **Step 1: 使用Edit工具替换处理流程章节**

将第98-129行（从"## 处理流程（防止超时）"到"### 4. 最终合并"段落结束）替换为新内容：

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
- 询问用户是否并行执行：
  - A) 是，并行处理所有章节
  - B) 否，按顺序逐个处理
  - C) 部分并行（指定哪些章节并行）

### 3. 执行阶段
**并行模式**：
- Spawn多个Agent，每个Agent处理一个章节Task
- 每个Agent独立完成：生成逐字稿 + 自检风格 + 保存文件

**顺序模式**：
- 主session逐个处理Task
- 每完成一个章节，更新Task状态

### 4. 失败处理
- 单章节失败：自动重试最多2次
- 超过重试次数：标记failed，记录失败原因到 `处理状态.json`
- 合并阶段提示用户处理失败章节：
  - A) 重试该章节
  - B) 跳过该章节，合并其他章节
  - C) 手动处理该章节

### 5. 合并阶段
- 所有章节完成后Spawn合并Agent
- 合并Agent职责：
  - 按章节顺序拼接
  - 风格一致性检查（称呼用语、语气）
  - 内容完整性检查（衔接流畅、无重复、无遗漏）
  - 添加整体结构（标题、目录、开篇、结束语）
- 输出文件：
  - `逐字稿_完整版.md`（最终文档）
  - `合并检查报告.md`（检查报告）
```

- [ ] **Step 2: 验证修改后的文件完整性**

使用Read工具读取修改后的SKILL.md，确认：
- 第98行开始是新章节"## 处理流程（并行优化版）"
- 后续章节（使用方式、示例）位置正确，未被删除
- 文件总行数变化合理（原约32行被替换为约45行）

- [ ] **Step 3: 提交更改**

```bash
cd D:/note/lecture-transcript-skill
git add skills/lecture-transcript/SKILL.md
git commit -m "$(cat <<'EOF'
feat: update workflow to parallel-optimized version

Replace sequential chapter processing with:
- Chapter detection and task creation
- Parallel/sequential execution options
- Failure handling with retry mechanism
- Merge agent for consistency checking

Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>
EOF
)"
```

Expected output: `[master xxxxxxx] feat: update workflow...`

---

## Self-Review Checklist

### Spec Coverage

| Spec Section | Task |
|--------------|------|
| Workflow Summary | Task 1 - 包含完整流程 |
| Chapter Detection & Display | Task 1 - 章节1.章节解析阶段 |
| Parallel Execution Decision | Task 1 - 章节2.任务创建阶段 |
| Chapter Agent Responsibilities | Task 1 - 章节3.执行阶段 |
| Failure Handling | Task 1 - 章节4.失败处理 |
| Merge Agent Responsibilities | Task 1 - 章节5.合并阶段 |
| SKILL.md Updates | Task 1 - 直接修改 |

**Coverage: 100%** - 所有spec内容已映射到Task 1。

### Placeholder Scan

- 无"TBD"、"TODO"、"implement later"等占位符
- 无"Add appropriate error handling"等模糊描述
- 所有代码块包含具体内容
- 所有命令包含具体参数和预期输出

### Type Consistency

- 无函数/方法签名定义，此为markdown skill文件更新
- 文件路径一致：`skills/lecture-transcript/SKILL.md`
- 行号范围一致：98-129

---

## Plan Complete

计划已完成，覆盖所有spec需求，无占位符，类型一致。