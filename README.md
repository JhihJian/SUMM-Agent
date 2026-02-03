# SUMM Agent

SUMM 是智能工作助理，基于 Claude Code CLI 实现，通过 Manager-Worker 架构管理任务执行。

## 核心功能

- **想法管理**：记录和组织待办事项
- **状态可视化**：通过 Markdown 看板呈现工作进度
- **任务委托**：将任务分配给独立的 Claude Code 会话执行

## 快速开始

### 1. 复制 SUMM 目录

将 `SUMM/` 目录复制到你的项目位置：

```bash
cp -r SUMM/ /path/to/your/project/
cd /path/to/your/project/
```

### 2. 启动 SUMM

在 SUMM 目录中启动 Claude Code CLI：

```bash
cd SUMM
claude
```

### 3. 使用指令

- `/todo [任务描述]` - 创建新任务并启动 Worker
- `/status` - 查看当前状态
- `/clean` - 清空工作台（异常恢复）

## 项目结构

```
SUMM/
├── CLAUDE.md          # 系统提示词（Manager/Worker 行为规则）
├── summ_plan.md       # 任务看板
├── .summ/
│   ├── active/        # 工作台（当前执行中的任务）
│   ├── daily/         # 每日流水账
│   └── archive/       # 历史任务归档
└── .claude/skills/    # SUMM Skills
    ├── summ-todo.skill
    ├── summ-status.skill
    └── summ-clean.skill
```

## 文档

- [产品需求文档 (PRD)](docs/SUMM%20Agent%20产品需求文档%20(PRD)%20v1.0%20MVP%20版.md)
- [静态页面展示规范](docs/static-page-spec.md)

## 工作模式

```
用户 → Manager (当前会话) → Worker (独立会话) → 修改代码
         ↓                      ↑
    更新看板              生成 journal.md / handoff.md
```

## 依赖

- Claude Code CLI
- SUMM-Daemon（进程管理守护服务）
