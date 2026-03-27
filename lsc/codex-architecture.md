# Codex 架构与能力解析

Codex 是一个强大的 AI 编程助手，其能力来自于精心设计的多层架构。本文档讲解其核心设计原理。

## 整体架构

```
┌─────────────────────────────────────────────────────────────┐
│                        用户界面层                            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   TUI (Rust) │  │    CLI       │  │   Browser    │      │
│  │  ratatui UI  │  │  命令行接口   │  │  Chrome扩展  │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                        核心引擎层                            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   Session    │  │ Conversation │  │   Streaming  │      │
│  │   管理       │  │   上下文     │  │   流式处理    │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                        工具执行层                            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  File System │  │   Command    │  │    Browser   │      │
│  │  文件操作     │  │  命令执行     │  │  浏览器自动化 │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │     Git      │  │     MCP      │  │  Search Web  │      │
│  │  版本控制     │  │  扩展协议     │  │  网络搜索     │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                      AI 模型接入层                           │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   OpenAI     │  │  DeepSeek    │  │   Custom     │      │
│  │   GPT 系列   │  │  智谱/Kimi   │  │  自定义提供商  │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
```

## 核心能力来源

### 1. 上下文管理 (Context Management)

**文件位置：** `code-rs/core/src/codex/session.rs`

Codex 通过以下方式维护丰富的上下文：

```rust
// 会话状态管理
pub struct Session {
    // 对话历史
    conversation_history: Vec<ResponseItem>,

    // 工作目录状态
    cwd: PathBuf,

    // Git 仓库信息
    git_info: Option<GitInfo>,

    // 模型配置
    model_family: ModelFamily,
}
```

**关键特性：**
- **自动压缩**：当对话过长时自动总结历史
- **智能截断**：根据模型上下文窗口动态调整
- **增量更新**：只发送变更部分，减少 token 消耗

### 2. 工具调用系统 (Tool Calling)

**文件位置：** `code-rs/core/src/codex/streaming.rs`

Codex 通过 OpenAI 兼容的 Function Calling 实现工具调用：

```rust
// 工具定义
get_openai_tools(
    &tools_config,
    Some(mcp_tools),
    browser_enabled,
    agents_active,
    dynamic_tools,
)
```

**内置工具：**
- `read_file` - 读取文件内容
- `write_file` - 写入文件
- `list_files` - 列出目录
- `search_files` - 搜索文件内容
- `run_command` - 执行 shell 命令
- `git_diff` - 获取 Git 差异
- `browser_snapshot` - 浏览器快照

### 3. Auto Drive 自主执行

**文件位置：** `code-rs/code-auto-drive-core/src/auto_coordinator.rs`

这是 Codex 最强大的功能之一，实现完全自主的任务执行：

```rust
pub struct AutoCoordinator {
    // 状态机
    state: AutoResolveState,

    // 执行历史
    history: AutoDriveHistory,

    // 错误处理
    fault_handler: FaultHandler,
}
```

**执行流程：**

```
用户输入
    ↓
分析任务 → 创建执行计划
    ↓
执行步骤 → 调用工具 → 观察结果
    ↓
    ├─ 成功 → 继续下一步
    ├─ 失败 → 重试/调整策略
    └─ 需要确认 → 等待用户反馈
    ↓
任务完成 → 生成报告
```

**关键能力：**
- **智能重试**：失败后自动调整策略
- **错误恢复**：从错误中学习并修正
- **进度可视化**：实时显示执行进度
- **可中断**：用户随时可以暂停/停止

### 4. 沙箱执行 (Sandbox)

**文件位置：** `code-rs/linux-sandbox/src/landlock.rs`

使用 Linux Landlock 实现安全的沙箱：

```rust
pub enum SandboxMode {
    ReadOnly,              // 只读
    WorkspaceWrite,        // 工作区可写
    DangerFullAccess,      // 完全访问（危险）
}
```

**安全特性：**
- 文件系统访问控制
- 网络访问限制
- 命令执行审批
- 环境变量过滤

### 5. 模型抽象层

**文件位置：** `code-rs/core/src/model_family.rs`

统一的模型接口，支持多种 AI 提供商：

```rust
pub fn find_family_for_model(slug: &str) -> Option<ModelFamily> {
    if slug.starts_with("gpt-5") {
        // OpenAI GPT-5 系列
    } else if slug.starts_with("deepseek") {
        // DeepSeek 系列
    } else if slug.starts_with("glm") {
        // 智谱 GLM 系列
    }
    // ...
}
```

**优势：**
- 统一接口：不同模型使用相同 API
- 能力映射：根据模型能力调整功能
- 降级策略：模型不支持时自动降级

### 6. TUI 实时渲染

**文件位置：** `code-rs/tui/src/chatwidget.rs`

使用 ratatui 构建的终端 UI：

```rust
// 流式渲染
fn render_streaming_content(
    &mut self,
    content: &str,
    order_key: OrderKey,
) {
    // 实时显示 AI 输出
    // 支持 Markdown 高亮
    // 代码语法高亮
}
```

**特性：**
- 流式显示：AI 回复逐字显示
- Markdown 渲染：支持格式化文本
- 代码高亮：语法高亮显示
- 历史记录：可回溯之前的对话

## 工作流程示例

### 用户请求："帮我写一个猜数字游戏"

```
1. 解析意图
   └→ 识别为编程任务

2. 规划步骤
   ├→ 创建项目结构
   ├→ 编写游戏逻辑
   ├→ 添加用户交互
   └→ 测试功能

3. 执行步骤
   ├→ read_file("src/main.rs")  # 检查现有文件
   ├→ write_file("src/main.rs", game_code)  # 写入代码
   ├→ run_command("cargo run")  # 运行测试
   └→ 分析输出，确认成功

4. 返回结果
   └→ "游戏已创建并测试通过！"
```

## 关键设计模式

### 1. 事件驱动架构

```rust
pub enum EventMsg {
    Text(TextEvent),
    Error(ErrorEvent),
    Warning(WarningEvent),
    ToolCall(ToolCallEvent),
    // ...
}
```

所有操作通过事件传递，解耦组件。

### 2. 状态机

Auto Drive 使用状态机管理执行流程：

```rust
pub enum AutoResolveState {
    Planning,
    Executing,
    WaitingForApproval,
    Completed,
    Failed,
}
```

### 3. 策略模式

不同的模型使用不同的策略：

```rust
pub trait ModelStrategy {
    fn context_window(&self) -> usize;
    fn supports_tool_calling(&self) -> bool;
    fn max_tokens(&self) -> usize;
}
```

## 扩展性

### 添加新工具

1. 定义工具接口
2. 实现工具逻辑
3. 注册到工具系统

### 添加新模型

1. 在 `model_family.rs` 添加模型族
2. 在 `model_presets.rs` 添加模型预设
3. 构建更新

### 添加新功能

模块化设计允许独立开发和测试新功能。

## 总结

Codex 的强大来自于：

1. **丰富的上下文** - 理解项目结构和历史
2. **工具调用** - 能执行实际操作
3. **自主执行** - Auto Drive 自动完成复杂任务
4. **安全沙箱** - 保护系统安全
5. **模型抽象** - 支持多种 AI 模型
6. **实时反馈** - TUI 提供即时可视化

这种设计使 Codex 不仅能"对话"，更能"做事"。
