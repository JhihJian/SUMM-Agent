---
name: summ-todo
description: SUMM Manager 的任务创建技能。当用户提交 /todo 指令时触发：创建新任务、生成任务文件、启动 Worker Session。用于将任务委托给独立的 Claude Code 会话执行。
---

# SUMM Todo

## 工作流程

当用户使用 `/todo [任务描述]` 提交任务时，按以下步骤执行：

### 1. 生成任务 ID

格式：`T_YYYYMMDD_HHMMSS`

```bash
TASK_ID="T_$(date +%Y%m%d_%H%M%S)"
```

### 2. 更新任务看板

在 `summ_plan.md` 中添加任务条目：

```markdown
## 进行中

- [Working] T_<id> - <任务描述摘要>
```

### 3. 判断工作空间

根据任务内容判断使用哪种工作空间：

- **当前项目目录**：任务涉及修改现有代码/文件
- **临时目录**：`/tmp/summ-workspace-<id>/`（全新任务、调研类任务）

不确定时询问用户。

### 4. 创建任务文件

创建 `.summ/active/task.md`：

```markdown
# 任务 T_<id>

**描述**: <完整任务描述>
**创建时间**: <ISO 8601 格式时间>
**工作空间**: <工作空间路径>
**任务ID**: T_<id>
```

### 5. 启动 Worker Session

```bash
summ start --cli "claude" --init <工作空间> --name "worker-<任务ID>"
```

记录返回的 `session_id`。

### 6. 注入任务消息

```bash
summ inject <session_id> --message "你是 SUMM Worker。任务ID: T_<id>。任务描述: <任务描述>。请在工作空间中完成任务，完成后在工作空间根目录生成 journal.md 和 handoff.md 文件。"
```

### 7. 保存 Session 信息

将 `session_id` 保存到 `.summ/active/session.txt` 供后续状态查询。

## 任务监控

定期检查 Worker 状态：

```bash
# 检查 Session 是否存在
summ status <session_id>

# 检查是否生成了 handoff.md
ls <工作空间>/handoff.md
```

## 任务完成处理

当 Worker 完成任务后：

1. 从工作空间读取 `journal.md` 和 `handoff.md`
2. 将文件归档到 `.summ/archive/`：
   - `YYYYMMDD_<任务ID>_journal.md`
   - `YYYYMMDD_<任务ID>_handoff.md`
3. 更新 `summ_plan.md` 任务状态为 `[x]`
4. 停止 Worker Session：`summ stop <session_id>`
5. 清理临时工作空间（如适用）
6. 清空 `.summ/active/` 目录
