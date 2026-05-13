# Claude Multi-Window Manager (claude-mgr)

在一个终端窗口中管理多个 Claude Code 会话：分配任务、查看进度、发送消息、持久化上下文。

## 安装

### 依赖

- **tmux** ≥ 3.0 — 脚本首次运行 `claude-mgr setup` 时自动编译安装到 `~/.local/bin/tmux`
- **Claude Code CLI** (`claude`) — 需预先安装

### 添加到 PATH

```bash
# ~/.zshrc
export PATH="$HOME/.local/bin:$HOME/MyCode/scripts/pic:$PATH"
```

```bash
source ~/.zshrc
chmod +x ~/MyCode/scripts/pic/claude-mgr
```

## 快速开始

```bash
# 创建一个会话
claude-mgr new backend --task "用 FastAPI 写用户登录接口" --dir ~/projects/api

# 查看所有会话
claude-mgr ls

# 进入会话交互
claude-mgr attach backend

# 从外部发送指令
claude-mgr send backend "跑一下测试"

# 打开实时仪表盘
claude-mgr dashboard
```

## 命令参考

### 会话管理

| 命令 | 说明 |
|------|------|
| `claude-mgr new <name>` | 创建新会话 |
| `claude-mgr stop <name>` | 停止会话（保留上下文记录） |
| `claude-mgr resume <name>` | 恢复已停止的会话，自动接上之前的对话 |
| `claude-mgr delete <name>` | 永久删除会话及其记录 |

### 交互

| 命令 | 说明 |
|------|------|
| `claude-mgr attach <name>` | 切换到会话的 tmux 窗口，直接交互 |
| `claude-mgr send <name> <msg>` | 向会话发送消息（无需切入窗口） |

### 查看

| 命令 | 说明 |
|------|------|
| `claude-mgr ls` | 列出所有会话及运行状态 |
| `claude-mgr info <name>` | 查看会话详情 + 最近终端输出 |
| `claude-mgr log <name>` | 查看会话的完整终端输出 |
| `claude-mgr dashboard` | 实时仪表盘（自动刷新） |

### 任务与上下文

| 命令 | 说明 |
|------|------|
| `claude-mgr task <name> "描述"` | 设置或更新会话任务 |
| `claude-mgr context <name> --inject "..."` | 向运行中的会话注入上下文 |
| `claude-mgr import <name> --session-id <id>` | 导入已有的 Claude 会话 |

### 其他

| 命令 | 说明 |
|------|------|
| `claude-mgr setup` | 检查/安装依赖 |
| `claude-mgr export` | 导出所有会话数据为 JSON |

## 常用选项

### `new` 命令

```
claude-mgr new <name> [选项]

  --dir, -d <path>      工作目录（默认当前目录）
  --task, -t <text>     任务描述
  --model, -m <model>   Claude 模型（sonnet/opus/haiku）
  --effort, -e <level>  努力级别（low/medium/high/xhigh/max）
  --session-id <uuid>   恢复指定的 Claude 会话 ID
  --resume-from <name>  按名称恢复已有的 Claude 会话
```

### `send` 命令

```
claude-mgr send <name> <message...>

  --no-enter, -n    只输入文本，不按回车（用于撰写长消息）
```

### `resume` 命令

```
claude-mgr resume <name>

  --fresh, -f    全新启动，不恢复之前的对话
```

### `list` 命令

```
claude-mgr ls

  --all, -a    同时显示未被管理的 Claude 会话
```

### `log` 命令

```
claude-mgr log <name>

  --tail, -n <N>    只显示最后 N 行
```

### `delete` 命令

```
claude-mgr delete <name>

  --force, -f    强制删除（即使会话仍在运行）
```

### `dashboard` 命令

```
claude-mgr dashboard

  --refresh, -r <秒>    刷新间隔（默认 3 秒）
```

## 典型工作流

### 日常使用

```bash
# 早上开始工作
claude-mgr new api --task "完成用户模块的 CRUD 接口" --dir ~/projects/backend
claude-mgr new ui  --task "对接用户接口，写前端页面"  --dir ~/projects/frontend

# 查看状态
claude-mgr ls

# 给指定窗口发任务
claude-mgr send api "先写 User model 和数据库迁移"

# 过一会儿查看进展
claude-mgr log api --tail 20

# 切进去详细交互
claude-mgr attach api

# 下班前暂停
claude-mgr stop api
claude-mgr stop ui

# 第二天恢复（自动接上昨天的对话）
claude-mgr resume api
claude-mgr resume ui
```

### 仪表盘

```bash
claude-mgr dashboard
```

显示效果：

```
╔══════════════════════════════════════════════════════════════════════════════╗
║ Claude Multi-Window Manager                                       13:45:02 ║
╠══════════════════════════════════════════════════════════════════════════════╣
║ ● backend              w1   完成用户模块的 CRUD 接口         ~/projects/backend    2m ago ║
║   │ ❯ Run /init to create a CLAUDE.md file                                     ║
║   │ ⏺ I'll start by creating the User model...                                 ║
║ ● frontend             w2   对接用户接口，写前端页面         ~/projects/frontend   5m ago ║
║   │ ❯ what approach for API client?                                            ║
║   │ ⏺ I recommend using fetch with a custom hook...                            ║
╠══════════════════════════════════════════════════════════════════════════════════╣
║ ● 2 running  ○ 0 stopped  refresh: 3s  [q]uit [r]efresh [n]ew [a]ttach       ║
╚══════════════════════════════════════════════════════════════════════════════════╝
```

按键操作：`q` 退出，`r` 刷新，`n` 新建，`a` 进入。

### 恢复已有的 Claude 会话

```bash
# 查看所有 Claude 会话（包括未被管理的）
claude-mgr ls --all

# 导入指定会话
claude-mgr import my-session --from-name "backend work"
claude-mgr resume my-session
```

## 架构

```
                    ┌─────────────────────────────────┐
                    │        claude-mgr (Python)        │
                    │     CLI 控制 / JSON 持久化        │
                    └──────────────┬──────────────────┘
                                   │ tmux commands
                    ┌──────────────▼──────────────────┐
                    │     tmux session: claude-mgr     │
                    │  ┌─────────┐  ┌─────────┐       │
                    │  │  w1: api │  │  w2: ui  │ ...  │
                    │  │ claude   │  │ claude   │       │
                    │  └─────────┘  └─────────┘       │
                    └─────────────────────────────────┘
```

- **后端**: tmux，管理终端多窗口
- **会话存储**: `~/.claude-mgr/sessions.json`，记录每个会话的名称、目录、任务、Claude session ID
- **上下文持久化**: 通过 `claude --resume <session-id>` 在窗口重启时自动恢复对话
- **输出捕获**: `tmux capture-pane` 获取每个窗口的终端内容

## 数据文件

| 文件 | 说明 |
|------|------|
| `~/.claude-mgr/sessions.json` | 会话注册表（名称、目录、任务、session ID） |
| `~/.claude-mgr/mgr.log` | 操作日志 |

## 卸载

```bash
rm -rf ~/.claude-mgr          # 删除所有会话数据
rm ~/.local/bin/tmux           # 删除本地编译的 tmux
rm ~/MyCode/scripts/pic/claude-mgr  # 删除脚本
```
