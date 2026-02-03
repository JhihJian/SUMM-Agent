---
name: summ-status
description: SUMM Manager 的状态查询技能。当用户提交 /status 指令时触发：显示任务看板内容、查询 Worker Session 运行状态。
---

# SUMM Status

## 工作流程

当用户使用 `/status` 查询状态时，按以下步骤执行：

### 1. 读取任务看板

读取并显示 `summ_plan.md` 的内容。

### 2. 查询 Worker Session 状态

执行以下命令查询 Worker 状态：

```bash
# 查询所有 Sessions
summ list

# 查询运行中的 Workers
summ list --status running

# 查询空闲的 Workers
summ list --status idle
```

### 3. 读取当前任务信息

如果 `.summ/active/task.md` 存在，显示当前执行的任务：
- 任务 ID
- 任务描述
- 创建时间
- 工作空间路径

### 4. 显示 Session 信息

如果 `.summ/active/session.txt` 存在，查询该 Session 的详细状态：

```bash
summ status $(cat .summ/active/session.txt)
```

### 5. 格式化输出

按以下格式输出状态：

```
## SUMM 状态

### 任务看板
[summ_plan.md 内容]

### 当前任务
[.summ/active/task.md 内容]

### Worker Sessions
[summ list 输出]
```
