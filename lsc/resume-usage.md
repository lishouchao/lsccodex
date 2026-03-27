# 会话恢复 (Resume) 使用指南

Codex 会自动保存所有会话，即使被强制终止也能恢复。

## 查看会话恢复提示

当 TUI 被强制终止（如 Ctrl+C 或关闭终端）时，会显示恢复信息：

```
To continue this session, run jecode resume e0008eb1-d6fc-40f8-99e6-feb7d88a9450
```

## 恢复会话的方法

### 1. 恢复指定会话

使用会话 ID 恢复特定会话：

```bash
jecode resume e0008eb1-d6fc-40f8-99e6-feb7d88a9450
```

### 2. 恢复最近的会话

直接恢复最近的会话，不显示选择器：

```bash
jecode resume --last
```

### 3. 从选择器恢复

显示所有历史会话，选择要恢复的会话：

```bash
jecode resume
```

## 恢复后会发生什么

- ✅ 所有之前的对话历史都会保留
- ✅ 上下文（文件、项目状态）都会恢复
- ✅ 可以继续之前未完成的任务
- ✅ 模型记得之前的讨论内容

## 会话存储位置

会话数据保存在：
```
~/.code/sessions/
├── index/
└── 2026/
    └── 03/
        └── 27/
            └── <会话ID>/
```

## 实用技巧

### 快速恢复最近会话

创建别名（添加到 `~/.bashrc` 或 `~/.zshrc`）：

```bash
alias jecode-last='jecode resume --last'
```

### 恢复时继续任务

恢复会话后可以直接继续之前的对话：

```bash
# 恢复会话
jecode resume --last

# 恢复后直接输入新任务
> 继续刚才的工作
```

### 查看所有会话

```bash
ls -la ~/.code/sessions/2026/03/27/
```

## 命令参考

```
Usage: jecode resume [OPTIONS] [SESSION_ID] [PROMPT]

Arguments:
  [SESSION_ID]    会话 ID (UUID)
  [PROMPT]        可选的起始提示词

Options:
  --last          恢复最近的会话（不显示选择器）
  -m, --model     指定模型
  -C, --cd <DIR>  指定工作目录
```

## 注意事项

1. **会话持久化**：所有对话自动保存，无需手动操作
2. **跨终端**：可以在不同终端恢复同一会话
3. **上下文保留**：恢复后所有上下文信息完整保留
4. **多会话**：可以同时存在多个会话，通过 ID 区分
