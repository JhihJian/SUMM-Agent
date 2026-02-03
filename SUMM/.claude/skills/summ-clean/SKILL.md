---
name: clean
description: SUMM Manager 的工作台清理技能。当用户提交 /clean 指令时触发：清空 .summ/active/ 目录，用于 Worker 异常终止后的手动恢复。
---

# SUMM Clean

## 工作流程

当用户使用 `/clean` 清理工作台时，按以下步骤执行：

### 1. 检查工作台状态

检查 `.summ/active/` 目录是否存在文件：

```bash
ls -la .summ/active/
```

### 2. 显示当前工作台内容

如果存在文件，列出将被删除的文件：
- `task.md` - 任务说明
- `journal.md` - 执行日志
- `handoff.md` - 交付报告
- `session.txt` - Session ID

### 3. 确认清理

提示用户确认是否清理：

```
工作台包含以下文件：
[文件列表]

是否确认清理？(yes/no)
```

### 4. 停止相关 Worker（可选）

如果 `.summ/active/session.txt` 存在，询问是否停止对应的 Worker：

```bash
SESSION_ID=$(cat .summ/active/session.txt)
summ stop $SESSION_ID
```

### 5. 清空工作台

删除 `.summ/active/` 下所有文件（保留 `.gitkeep`）：

```bash
find .summ/active/ -type f -delete
```

### 6. 确认完成

显示清理完成的确认信息。
