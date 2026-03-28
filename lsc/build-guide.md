# 编译指南

本文档介绍如何在本地编译 Every Code 项目。

## 目录结构说明

| 目录 | 用途 | Crate 前缀 | 可修改 |
|------|------|-----------|--------|
| **codex-rs/** | 上游镜像 (openai/codex) | `codex-*` | ❌ 只读 |
| **code-rs/** | 活跃开发目录 | `code-*` | ✅ 所有修改在这里 |
| **codex-cli/** | NPM 发布包封装 | - | 发布配置 |

---

## Windows 环境编译

### 1. 安装 Rust 工具链

```powershell
# 方法1: 使用 winget
winget install Rustlang.Rustup

# 方法2: 从官网下载
# 访问 https://rustup.rs/ 下载 rustup-init.exe
```

安装完成后，**重启终端或 VSCode** 使环境变量生效。

### 2. 验证安装

```powershell
rustup --version
rustc --version
cargo --version
```

如果提示 `'cargo' is not recognized`，手动添加 PATH：

```powershell
$env:PATH += ";C:\Users\$env:USERNAME\.cargo\bin"
```

### 3. 编译项目

```powershell
cd d:\Codex\lsccodex\code-rs
cargo build --profile dev-fast --bin code
```

如果 `dev-fast` profile 不存在，使用：

```powershell
cargo build --release --bin code
```

### 4. 编译产物

编译完成后，可执行文件位于：

```
code-rs\target\dev-fast\code.exe    # dev-fast 模式
code-rs\target\release\code.exe     # release 模式
```

---

## Linux 环境编译

### 1. 安装 Rust 工具链

```bash
# 使用 rustup 安装
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# 加载环境变量
source $HOME/.cargo/env
```

或者使用包管理器：

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install -y curl build-essential pkg-config libssl-dev
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# Fedora
sudo dnf install -y curl gcc openssl-devel
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# Arch Linux
sudo pacman -S --needed curl base-devel openssl
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

### 2. 验证安装

```bash
rustup --version
rustc --version
cargo --version
```

### 3. 编译项目

**方法一：使用 build-fast.sh 脚本（推荐）**

```bash
cd /path/to/lsccodex
./build-fast.sh
```

脚本参数：

```bash
# 指定编译 profile
PROFILE=release ./build-fast.sh

# 编译后直接运行
./build-fast.sh run

# 查看详细环境信息
TRACE_BUILD=1 ./build-fast.sh

# 编译多个二进制文件
BUILD_FAST_BINS="code code-tui" ./build-fast.sh

# 性能分析版本
./build-fast.sh perf
```

**方法二：手动编译**

```bash
cd /path/to/lsccodex/code-rs
cargo build --profile dev-fast --bin code
```

### 4. 编译产物

```bash
code-rs/target/dev-fast/code      # dev-fast 模式
code-rs/target/release/code       # release 模式
./bin/code                         # 脚本自动复制的快捷方式
```

---

## macOS 环境编译

### 1. 安装依赖

```bash
# 安装 Xcode 命令行工具
xcode-select --install

# 安装 Homebrew（如果没有）
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

### 2. 安装 Rust 工具链

```bash
# 使用 rustup 安装
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# 加载环境变量
source $HOME/.cargo/env
```

或者使用 Homebrew：

```bash
brew install rustup
rustup-init
```

### 3. 验证安装

```bash
rustup --version
rustc --version
cargo --version
```

### 4. 编译项目

**方法一：使用 build-fast.sh 脚本（推荐）**

```bash
cd /path/to/lsccodex
./build-fast.sh
```

脚本参数：

```bash
# 指定编译 profile
PROFILE=release ./build-fast.sh

# 编译后直接运行
./build-fast.sh run

# 查看详细环境信息
TRACE_BUILD=1 ./build-fast.sh

# 确定性构建（移除 UUID，用于发布）
DETERMINISTIC=1 DETERMINISTIC_NO_UUID=1 ./build-fast.sh
```

**方法二：手动编译**

```bash
cd /path/to/lsccodex/code-rs
cargo build --profile dev-fast --bin code
```

### 5. 编译产物

```bash
code-rs/target/dev-fast/code           # Intel Mac (x86_64)
code-rs/target/dev-fast/code           # Apple Silicon (aarch64)
./bin/code                              # 脚本自动复制的快捷方式
```

### 6. Apple Silicon 注意事项

如果你使用 M1/M2/M3 等 Apple Silicon 芯片：

```bash
# 确认架构
uname -m
# 输出: arm64

# 如果需要为 Intel Mac 交叉编译
rustup target add x86_64-apple-darwin
cargo build --profile dev-fast --bin code --target x86_64-apple-darwin
```

---

## Profile 说明

| Profile | 用途 | 优化级别 | 编译时间 |
|---------|------|---------|---------|
| `dev-fast` | 日常开发 | 低优化 | 快 |
| `dev` | 调试 | 无优化 | 最快 |
| `release` | 发布 | 最高优化 | 慢 |
| `release-prod` | 生产发布 | 最高优化 + LTO | 最慢 |
| `perf` | 性能分析 | 保留 debug 符号 | 慢 |

---

## 常见问题

### 1. Cargo.lock 不一致

```
⚠️  Warning: Cargo.lock appears out of date or inconsistent
```

解决方法：

```bash
cargo update
```

### 2. 编译缓存问题

如果遇到奇怪的编译错误，尝试清理缓存：

```bash
cargo clean
cargo build --profile dev-fast --bin code
```

### 3. 内存不足

编译过程可能占用大量内存，如果失败可以尝试：

```bash
# Linux/macOS
export CARGO_BUILD_JOBS=2
cargo build --profile dev-fast --bin code

# Windows PowerShell
$env:CARGO_BUILD_JOBS = "2"
cargo build --profile dev-fast --bin code
```

### 4. OpenSSL 链接错误 (Linux)

```bash
# Ubuntu/Debian
sudo apt install -y libssl-dev pkg-config

# Fedora
sudo dnf install -y openssl-devel

# Arch
sudo pacman -S openssl
```

### 5. 没有 rustup

```bash
# Linux/macOS
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source $HOME/.cargo/env
```

---

## 开发工作流

1. **修改代码** - 只修改 `code-rs/` 目录下的文件
2. **编译验证** - 运行 `cargo build --profile dev-fast`
3. **运行测试** - 运行 `cargo test` (可选)
4. **提交代码** - 遵循 Conventional Commits 规范

### 运行测试

```bash
# 运行所有测试
cargo nextest run --no-fail-fast

# 运行特定包的测试
cargo test -p code-tui --features test-helpers
cargo test -p code-cloud-tasks --tests

# 运行 VT100 快照测试
cargo test -p code-tui --test vt100_chatwidget_snapshot --features test-helpers
```
