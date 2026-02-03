# SUMM Agent - 产品需求文档 (PRD)

**版本:** 1.0 (MVP)
**日期:** 2026年2月

---

## 1. 项目概述

### 1.1 核心价值

SUMM 是智能工作助理，核心功能：
- **想法管理**：记录和组织待办事项
- **状态可视化**：通过 Markdown 看板呈现工作进度
- **任务委托**：将任务分配给独立的 Claude Code 会话执行

SUMM 的价值在于维护工作秩序，而非执行具体任务。

### 1.2 工作模式

- **用户**：提出需求
- **Manager**：记录、组织、委托任务
- **Worker**：独立 Claude Code 会话，执行具体任务

每个任务启动一个独立的 Worker 会话。

### 1.3 设计原则

1. **Markdown 存储**：所有状态和结果以 Markdown 文件形式存储
2. **目录隔离**：管理文件存放于 `.summ/` 目录，保持项目根目录整洁
3. **手动干预**：错误不自动重试，由用户决定后续操作

### 1.4 技术依赖

- Claude Code CLI
- SUMM-Daemon（进程管理守护服务，负责启动和管理 Worker Session）

---

## 2. 目录结构

采用"工作台 + 扁平归档"的极简结构：

**根目录文件：**
- `CLAUDE.md` - 系统提示词，定义 Manager 和 Worker 的行为规则
- `summ_plan.md` - 全局计划看板，用户主要交互界面

**`.summ/` 隐藏目录：**
- `active/` - 工作台，存放当前执行任务的文件（任务结束即清空）
  - `task.md` - Manager 下发的任务书
  - `journal.md` - Worker 记录的执行流水
  - `handoff.md` - Worker 提交的交付契约
- `daily/` - 每日流水账（按月归档，如 `2026-02.md`）
- `archive/` - 历史任务归档（扁平化存储，文件名格式：`YYYYMMDD_任务标识_类型.md`）


---

## 3. 工作机制

### 3.1 任务处理

用户提交任务后，Manager 执行以下操作：
- 记录任务至 `summ_plan.md`
- 生成任务说明文件
- 启动独立 Worker 会话执行任务
- 任务完成后更新状态并归档

### 3.2 任务粒度

任务粒度由用户自主决定，可以是：
- 完整功能实现
- 局部代码修改
- 技术方案调研

Manager 不对任务进行拆分或重组。

---

## 4. 工作流程

### 4.1 任务执行流程

**任务启动**
1. 用户通过 `/todo` 指令提交任务描述
2. Manager 在 `summ_plan.md` 记录任务
3. Manager 生成 `task.md` 任务说明文件
4. Manager 通过 SUMM-Daemon 启动 Worker 会话

**任务执行**
1. Worker 读取 `task.md` 获取任务要求
2. Worker 执行任务，修改项目文件
3. Worker 在 `journal.md` 记录关键决策
4. Worker 生成 `handoff.md` 交付报告

**任务收尾**
1. Manager 读取 `handoff.md` 获取执行结果
2. Manager 更新 `summ_plan.md` 任务状态
3. Manager 归档工作文件至 `archive/`
4. Manager 终止 Worker 会话

### 4.2 角色定义

- **Manager**：当前 Claude Code 会话，负责任务管理和协调
- **Worker**：独立 Claude Code 会话，负责任务执行
- **SUMM-Daemon**：后台守护进程，管理 Worker 会话生命周期

---

## 5. 数据文件

### 5.1 用户界面文件

**`summ_plan.md`** - 任务看板
- `- [ ]` 待办
- `- [Working]` 进行中
- `- [x]` 已完成
- `- [Failed]` 失败

### 5.2 系统管理文件（`.summ/` 目录）

**工作台（`active/`）**
- `task.md` - 任务说明
- `journal.md` - 执行日志
- `handoff.md` - 交付报告

**归档（`archive/`）**
- 历史任务文件，按日期命名

**流水账（`daily/`）**
- 按月记录任务摘要

---

## 6. 用户指令

**`/todo [任务描述]`**
- 提交新任务
- Manager 记录任务并启动 Worker 会话

**`/status`**
- 查询当前工作状态
- 显示 `summ_plan.md` 内容及 Worker 运行状态

**`/clean`**
- 清空工作台
- 用于 Worker 异常时的手动恢复

---

## 7. 行为规则

通过 `CLAUDE.md` 定义 Manager 和 Worker 的行为规范。

**Manager 规则**
- 不直接修改项目代码，仅负责任务管理
- 任务失败时不自动重试，等待用户指令
- 任务完成后清空工作台并归档文件

**Worker 规则**
- 启动后立即读取 `task.md`
- 执行过程中记录关键决策至 `journal.md`
- 完成后生成 `handoff.md` 并退出

---

## 8. MVP 验收标准

1. 用户执行 `/todo` 后，系统生成 `task.md` 并启动 Worker 会话
2. Worker 执行任务后，生成 `handoff.md` 和 `journal.md`
3. 任务完成后，`summ_plan.md` 状态自动更新
4. 工作文件归档至 `.summ/archive/`，工作台清空
5. 每日摘要记录至 `.summ/daily/`
6. Manager 可通过 SUMM-Daemon 管理 Worker 会话生命周期

---

## 9. 异常处理

**Worker 异常终止**
- Manager 检测到 Worker 终止但未生成 `handoff.md`
- 标记任务为失败状态，保留工作台文件
- 用户通过 `/clean` 清理后可重新执行任务

**归档失败**
- 归档操作失败时保留工作台文件
- 记录错误信息
- 等待用户手动处理

