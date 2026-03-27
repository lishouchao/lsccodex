# 如何添加新模型到 Codex

本文档说明如何在 Codex 中添加新的 AI 模型支持。

## 概述

添加新模型需要修改两个核心文件：

1. **模型预设** (`code-rs/common/src/model_presets.rs`) - 定义模型的基本信息和显示
2. **模型族** (`code-rs/core/src/model_family.rs`) - 定义模型的行为和能力

## 步骤 1: 添加模型预设

编辑 `code-rs/common/src/model_presets.rs`，在 `PRESETS` 向量中添加新的 `ModelPreset`：

```rust
ModelPreset {
    id: "model-id".to_string(),              // 唯一标识符
    model: "model-name".to_string(),          // API 调用时的模型名称
    display_name: "Model Display Name".to_string(),  // UI 中显示的名称
    description: "Model description.".to_string(),   // 模型描述
    default_reasoning_effort: ReasoningEffort::Medium,  // 默认推理级别
    supported_reasoning_efforts: vec![        // 支持的推理级别
        ReasoningEffortPreset {
            effort: ReasoningEffort::Medium,
            description: "Balanced speed and reasoning".to_string(),
        },
    ],
    supported_text_verbosity: ALL_TEXT_VERBOSITY,  // 支持的文本详细度
    is_default: false,                             // 是否为默认模型
    upgrade: None,                                 // 升级路径（可选）
    pro_only: false,                               // 是否仅限 Pro 用户
    show_in_picker: true,                          // 是否在选择器中显示
},
```

### 字段说明

| 字段 | 说明 |
|------|------|
| `id` | 唯一标识符，用于内部引用 |
| `model` | API 调用时使用的模型名称 |
| `display_name` | UI 中显示的友好名称 |
| `description` | 模型的简短描述 |
| `default_reasoning_effort` | 默认推理级别：`None`, `Minimal`, `Low`, `Medium`, `High`, `XHigh` |
| `supported_reasoning_efforts` | 该模型支持的推理级别列表 |
| `supported_text_verbosity` | 支持的文本详细度：`ALL_TEXT_VERBOSITY` 或 `&[TextVerbosityConfig::Medium]` |
| `is_default` | 是否为默认模型（只能有一个） |
| `upgrade` | 可选，指向更新的模型版本 |
| `pro_only` | 是否仅对 ChatGPT Pro 用户开放 |
| `show_in_picker` | 是否在模型选择器中显示 |

## 步骤 2: 添加模型族（如需要）

如果新模型不属于任何现有的模型族，需要在 `code-rs/core/src/model_family.rs` 的 `find_family_for_model` 函数中添加：

```rust
} else if slug.starts_with("model-prefix") {
    // 模型族配置
    model_family!(
        slug, "model-family-name",
        supports_reasoning_summaries: true,
        base_instructions: BASE_INSTRUCTIONS.to_string(),
        apply_patch_tool_type: Some(ApplyPatchToolType::Freeform),
        supports_parallel_tool_calls: true,
        context_window: Some(CONTEXT_WINDOW_128K),
        max_output_tokens: Some(8_192),
        truncation_policy: TruncationPolicy::Bytes(10_000),
    )
```

### 模型族字段说明

| 字段 | 说明 |
|------|------|
| `supports_reasoning_summaries` | 是否支持推理摘要 |
| `base_instructions` | 基础系统提示词 |
| `apply_patch_tool_type` | 补丁工具类型：`None`, `Some(Freeform)`, `Some(Function)` |
| `supports_parallel_tool_calls` | 是否支持并行工具调用 |
| `context_window` | 上下文窗口大小 |
| `max_output_tokens` | 最大输出 token 数 |
| `truncation_policy` | 截断策略：`Tokens(n)` 或 `Bytes(n)` |

## 步骤 3: 构建验证

运行构建脚本验证更改：

```bash
./build-fast.sh
```

确保构建成功且无警告。

## 步骤 4: 更新二进制文件

```bash
# 关闭正在运行的 TUI
# 然后复制新二进制文件
cp .code/working/_target-cache/just-every-codex/main-0d6e4079e367-c123ee26fc6f/code-rs/dev-fast/code ./bin/code
```

## 步骤 5: 提交更改

```bash
git add code-rs/common/src/model_presets.rs
git add code-rs/core/src/model_family.rs  # 如果修改了
git commit -m "feat: add <model-name> model preset"
```

## 示例：添加 DeepSeek 模型

以下是添加 DeepSeek 系列模型的完整示例：

### 1. 模型族（已有 `deepseek` 前缀覆盖）

```rust
// code-rs/core/src/model_family.rs
} else if slug.starts_with("deepseek") {
    model_family!(
        slug, "deepseek",
        supports_reasoning_summaries: true,
        base_instructions: BASE_INSTRUCTIONS.to_string(),
        apply_patch_tool_type: Some(ApplyPatchToolType::Freeform),
        supports_parallel_tool_calls: true,
        context_window: Some(CONTEXT_WINDOW_128K),
        max_output_tokens: Some(8_192),
        truncation_policy: TruncationPolicy::Bytes(10_000),
    )
```

### 2. 模型预设

```rust
// code-rs/common/src/model_presets.rs

// deepseek-chat
ModelPreset {
    id: "deepseek-chat".to_string(),
    model: "deepseek-chat".to_string(),
    display_name: "DeepSeek Chat".to_string(),
    description: "High-performance Chinese language model with strong coding capabilities.".to_string(),
    default_reasoning_effort: ReasoningEffort::Medium,
    supported_reasoning_efforts: vec![
        ReasoningEffortPreset {
            effort: ReasoningEffort::Medium,
            description: "Balanced speed and reasoning for everyday tasks".to_string(),
        },
    ],
    supported_text_verbosity: ALL_TEXT_VERBOSITY,
    is_default: false,
    upgrade: None,
    pro_only: false,
    show_in_picker: true,
},

// deepseek-reasoner
ModelPreset {
    id: "deepseek-reasoner".to_string(),
    model: "deepseek-reasoner".to_string(),
    display_name: "DeepSeek V3 (Reasoner)".to_string(),
    description: "DeepSeek's advanced reasoning model for complex problem-solving.".to_string(),
    default_reasoning_effort: ReasoningEffort::High,
    supported_reasoning_efforts: vec![
        ReasoningEffortPreset {
            effort: ReasoningEffort::Medium,
            description: "Standard reasoning depth".to_string(),
        },
        ReasoningEffortPreset {
            effort: ReasoningEffort::High,
            description: "Extended reasoning for complex problems".to_string(),
        },
    ],
    supported_text_verbosity: ALL_TEXT_VERBOSITY,
    is_default: false,
    upgrade: None,
    pro_only: false,
    show_in_picker: true,
},

// deepseek-coder
ModelPreset {
    id: "deepseek-coder".to_string(),
    model: "deepseek-coder".to_string(),
    display_name: "DeepSeek Coder".to_string(),
    description: "DeepSeek's code-specialized model optimized for programming and development tasks.".to_string(),
    default_reasoning_effort: ReasoningEffort::Medium,
    supported_reasoning_efforts: vec![
        ReasoningEffortPreset {
            effort: ReasoningEffort::Medium,
            description: "Balanced speed and reasoning for coding tasks".to_string(),
        },
    ],
    supported_text_verbosity: ALL_TEXT_VERBOSITY,
    is_default: false,
    upgrade: None,
    pro_only: false,
    show_in_picker: true,
},
```

## 常量参考

以下是可用的常量：

```rust
// 上下文窗口大小
CONTEXT_WINDOW_16K
CONTEXT_WINDOW_96K
CONTEXT_WINDOW_128K
CONTEXT_WINDOW_200K
CONTEXT_WINDOW_272K
CONTEXT_WINDOW_1M

// 最大输出
MAX_OUTPUT_DEFAULT

// 系统提示词
BASE_INSTRUCTIONS
BASE_INSTRUCTIONS_WITH_APPLY_PATCH
GPT_5_CODEX_INSTRUCTIONS
GPT_5_1_CODEX_MAX_INSTRUCTIONS
GPT_5_2_CODEX_INSTRUCTIONS
GPT_5_2_INSTRUCTIONS
GPT_5_1_INSTRUCTIONS

// 文本详细度
ALL_TEXT_VERBOSITY  // &[Low, Medium, High]

// 推理级别
ReasoningEffort::None
ReasoningEffort::Minimal
ReasoningEffort::Low
ReasoningEffort::Medium
ReasoningEffort::High
ReasoningEffort::XHigh
```

## 注意事项

1. **唯一性**：每个模型的 `id` 和 `model` 字段必须唯一
2. **默认模型**：整个项目只能有一个 `is_default: true` 的模型
3. **模型族覆盖**：如果多个模型共享相同的行为，使用 `slug.starts_with("prefix")` 统一处理
4. **API 兼容性**：确保 `model` 字段与 API 提供商的模型名称完全匹配
5. **测试**：添加后务必在 TUI 中测试模型选择和调用功能
