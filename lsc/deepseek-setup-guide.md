# DeepSeek 模型配置指南

本文档介绍如何编译 Every Code 项目并配置 DeepSeek 模型。

---

## 目录

- [编译安装](#编译安装)
- [DeepSeek 模型配置](#deepseek-模型配置)
- [主题配置](#主题配置)
- [全局命令设置](#全局命令设置)
- [故障排除](#故障排除)

---

## 编译安装

### 1. 安装 Rust 工具链

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source $HOME/.cargo/env
```

### 2. 安装系统依赖

**Ubuntu/Debian:**

```bash
sudo apt update
sudo apt install -y build-essential libssl-dev pkg-config cmake curl git
```

**Fedora:**

```bash
sudo dnf install -y gcc openssl-devel pkg-config cmake curl git
```

**Arch Linux:**

```bash
sudo pacman -S --needed base-devel openssl cmake curl git
```

### 3. 编译项目

```bash
# 进入项目目录
cd /path/to/lsccodex

# 使用快速编译脚本
./build-fast.sh

# 或编译所有二进制文件
BUILD_FAST_BINS="code code-tui code-exec" ./build-fast.sh
```

**编译产物位置:**

```
./bin/code         # 主 CLI 工具
./bin/code-tui     # 交互式 TUI 界面
./bin/code-exec    # 无头执行模式
```

**编译时间:**
- 首次编译（冷缓存）: 20+ 分钟
- 后续编译（热缓存）: 1-5 分钟

---

## DeepSeek 模型配置

### 关键配置要点

**重要说明:** 配置文件中的 `api_key` 字段**不会被应用检查**。必须使用以下两种方式之一：

| 方式 | 字段名 | 安全性 | 推荐度 |
|------|--------|--------|--------|
| 环境变量 | `env_key` | ✅ 高 | ⭐⭐⭐⭐⭐ |
| 直接令牌 | `experimental_bearer_token` | ⚠️ 低 | ⭐⭐⭐ |

### 方式一: 环境变量（推荐）

**1. 配置文件** (`~/.code/config.toml`):

```toml
# DeepSeek 模型配置
model = "deepseek-chat"
model_provider = "deepseek"

# DeepSeek 提供商配置
[model_providers.deepseek]
name = "DeepSeek"
base_url = "https://api.deepseek.com/v1"
env_key = "DEEPSEEK_API_KEY"
wire_api = "chat"
requires_openai_auth = false
```

**2. 设置环境变量:**

```bash
# 临时设置（当前会话）
export DEEPSEEK_API_KEY="sk-your-api-key-here"

# 永久设置（添加到 ~/.bashrc 或 ~/.zshrc）
echo 'export DEEPSEEK_API_KEY="sk-your-api-key-here"' >> ~/.bashrc
source ~/.bashrc
```

### 方式二: 直接令牌

**配置文件** (`~/.code/config.toml`):

```toml
# DeepSeek 模型配置
model = "deepseek-chat"
model_provider = "deepseek"

# DeepSeek 提供商配置
[model_providers.deepseek]
name = "DeepSeek"
base_url = "https://api.deepseek.com/v1"
experimental_bearer_token = "sk-your-api-key-here"
wire_api = "chat"
requires_openai_auth = false
```

### 配置字段说明

| 字段 | 说明 | 必需 |
|------|------|------|
| `name` | 提供商显示名称 | ✅ |
| `base_url` | API 基础 URL | ✅ |
| `env_key` | 环境变量名 | 方式一必需 |
| `experimental_bearer_token` | API 令牌 | 方式二必需 |
| `wire_api` | API 协议类型 (`"chat"` 或 `"responses"`) | ✅ |
| `requires_openai_auth` | 是否需要 OpenAI 认证 | ✅ 必须设为 `false` |

### 可用的 DeepSeek 模型

| 模型 | 说明 | 配置值 |
|------|------|--------|
| DeepSeek Chat | 通用对话模型 | `deepseek-chat` |
| DeepSeek Reasoner | 高级推理模型 | `deepseek-reasoner` |
| DeepSeek Coder | 代码专用模型 | `deepseek-coder` |

---

## 主题配置

### 深色主题设置

在 `~/.code/config.toml` 中添加：

```toml
# TUI 主题配置
[tui.theme]
name = "dark-carbon-night"
```

### 可用的深色主题

| 主题名称 | 说明 |
|---------|------|
| `dark-carbon-night` | 默认深色主题，蓝紫色调 |
| `dark-oled-black-pro` | 纯黑背景，适合 OLED 显示器 |
| `dark-shinobi-dusk` | 日式暮色主题 |
| `dark-amber-terminal` | 琥珀色 CRT 复古风格 |
| `dark-aurora-flux` | 极光主题，青绿色调 |
| `dark-zen-garden` | 禅意花园，柔和色调 |

---

## 全局命令设置

### 方法一: 添加到 PATH（推荐）

```bash
# 添加到 ~/.bashrc 或 ~/.zshrc
echo 'export PATH="/path/to/lsccodex/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

### 方法二: 创建符号链接

```bash
# 创建个人 bin 目录
mkdir -p ~/bin
cd ~/bin

# 创建符号链接
ln -s /path/to/lsccodex/bin/code jecode
ln -s /path/to/lsccodex/bin/code-tui jecode-tui
ln -s /path/to/lsccodex/bin/code-exec jecode-exec

# 确保 ~/bin 在 PATH 中
echo 'export PATH="$HOME/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

### 使用方法

```bash
jecode          # 启动主程序
jecode-tui      # 启动交互式界面
jecode-exec     # 无头执行模式
```

---

## 故障排除

### 问题一: 仍然显示 "Sign in with ChatGPT"

**原因分析:**

应用的认证检查流程存在两个独立逻辑：

1. **早期检查** (`is_using_chatgpt_auth`): 仅检查 `auth.json`
2. **后期检查** (`get_login_status`): 优先尝试 ChatGPT 认证，失败后才检查自定义提供商

当使用 `api_key` 字段时，`has_custom_provider_api_key()` 函数无法识别，导致返回 `NotAuthenticated` 状态。

**解决方案:**

确保使用 `env_key` + 环境变量或 `experimental_bearer_token`：

```toml
# ❌ 错误配置
[model_providers.deepseek]
api_key = "sk-your-key"  # 这个字段不会被检查！

# ✅ 正确配置（方式一）
[model_providers.deepseek]
env_key = "DEEPSEEK_API_KEY"

# ✅ 正确配置（方式二）
[model_providers.deepseek]
experimental_bearer_token = "sk-your-key"
```

### 问题二: 编译失败 - OpenSSL 相关错误

**错误信息:**

```
Could not find directory of OpenSSL installation
```

**解决方案:**

```bash
# Ubuntu/Debian
sudo apt install -y libssl-dev pkg-config

# Fedora
sudo dnf install -y openssl-devel

# Arch Linux
sudo pacman -S openssl
```

### 问题三: 编译时间过长

**解决方案:**

```bash
# 使用 dev-fast profile（默认）
./build-fast.sh

# 限制并行编译任务数（低内存环境）
export CARGO_BUILD_JOBS=2
./build-fast.sh
```

### 问题四: 命令名冲突（与 VSCode）

**问题:** VSCode 也使用 `code` 命令

**解决方案:**

重命名二进制文件或使用别名：

```bash
cd /path/to/lsccodex/bin
mv code jecode
mv code-tui jecode-tui
mv code-exec jecode-exec
```

---

## 完整配置示例

`~/.code/config.toml`:

```toml
# Every Code 配置文件

# ============================================
# 模型配置
# ============================================
model = "deepseek-chat"
model_provider = "deepseek"

# DeepSeek 提供商
[model_providers.deepseek]
name = "DeepSeek"
base_url = "https://api.deepseek.com/v1"
experimental_bearer_token = "sk-your-api-key-here"
wire_api = "chat"
requires_openai_auth = false

# ============================================
# TUI 主题配置
# ============================================
[tui.theme]
name = "dark-carbon-night"

# ============================================
# 其他配置
# ============================================
# 审批策略
approval_policy = "on-request"

# 沙箱模式
sandbox_mode = "read-only"

# 历史记录
[history]
persistence = "save-all"
max-bytes = 10485760  # 10MB
```

---

## 技术细节

### 认证流程代码位置

- **配置加载**: `code-rs/core/src/config.rs:1261` - `is_using_chatgpt_auth()`
- **登录状态检查**: `code-rs/tui/src/lib.rs:1276` - `get_login_status()`
- **自定义提供商检查**: `code-rs/tui/src/lib.rs:1293` - `has_custom_provider_api_key()`
- **启动流程**: `code-rs/tui/src/app/init.rs:263` - 初始化逻辑

### 关键函数分析

**`has_custom_provider_api_key()` 函数只检查:**

1. `env_key` 字段 + 环境变量
2. `experimental_bearer_token` 字段

**不检查 `api_key` 字段！**

这就是为什么使用 `api_key` 配置会显示登录界面的根本原因。

---

## 参考资源

- **项目地址**: [just-every/code](https://github.com/just-every/code)
- **上游项目**: [openai/codex](https://github.com/openai/codex)
- **DeepSeek API**: [https://api.deepseek.com](https://api.deepseek.com)
- **配置文档**: [./build-guide.md](./build-guide.md)
