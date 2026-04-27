# Gemini CLI Command Reference

Gemini CLI v0.16.0+ 完整参考，已更新模型版本并加入代理环境适配。

## 安装

```bash
npm install -g @google/gemini-cli
# Or without installing:
npx @google/gemini-cli
```

## 认证

```bash
# Option 1: API Key
export GEMINI_API_KEY=your_key

# Option 2: OAuth (interactive)
gemini  # First run prompts for auth
```

## 代理环境配置

### 检查当前代理

```bash
echo "https_proxy=$https_proxy http_proxy=$http_proxy"
```

### 推荐：配置 Shell Alias

为避免每个命令都手写代理变量，建议配置 alias：

```bash
# 添加到 ~/.zshrc 或 ~/.bashrc
alias pgemini='https_proxy=http://127.0.0.1:7890 http_proxy=http://127.0.0.1:7890 gemini'
alias pgemini-flash='https_proxy=http://127.0.0.1:7890 http_proxy=http://127.0.0.1:7890 gemini -m gemini-3-flash-preview --skip-trust'
alias pgemini-pro='https_proxy=http://127.0.0.1:7890 http_proxy=http://127.0.0.1:7890 gemini -m gemini-3.1-pro-preview --skip-trust'

# 后续命令简化为：
pgemini-pro -p "提示词"
pgemini-flash -p "提示词"  # 快速模型
```

### 代理地址参考

| 代理软件 | 默认端口 |
|---------|---------|
| Clash | 7890 |
| V2RayN | 10809 |
| Shadowsocks | 1080 |

> 根据实际代理软件调整 alias 中的端口

### 单命令代理设置（临时）

如未配置 alias，可单命令设置代理：

```bash
# Clash 默认端口
https_proxy=http://127.0.0.1:7890 http_proxy=http://127.0.0.1:7890 gemini [命令]

# V2RayN 默认端口
https_proxy=http://127.0.0.1:10809 http_proxy=http://127.0.0.1:10809 gemini [命令]
```

### 全局代理设置（可选）

如果终端已有全局代理，无需额外设置。设置全局代理：

```bash
# 临时代理（当前终端会话）
export https_proxy=http://127.0.0.1:7890
export http_proxy=http://127.0.0.1:7890

# 永久代理（写入 ~/.zshrc 或 ~/.bashrc）
echo 'export https_proxy=http://127.0.0.1:7890' >> ~/.zshrc
echo 'export http_proxy=http://127.0.0.1:7890' >> ~/.zshrc
```

> **注意**：全局代理会影响所有需要网络的命令。推荐使用 alias 方式，只对 Gemini 命令生效。

## 命令行参数

### 核心参数

| 参数 | 短写 | 描述 |
|------|------|------|
| `--yolo` | `-y` | 自动批准所有工具调用 |
| `--skip-trust` | | 跳过信任检查 |
| `--output-format` | `-o` | 输出格式：`text`, `json`, `stream-json` |
| `--model` | `-m` | 模型选择 |
| `--prompt` | `-p` | 非交互模式的提示词 |

### 模型选择（已更新）

| 模型 | 用途 | 上下文 |
|------|------|--------|
| `gemini-3.1-pro-preview` | 复杂任务（首选） | 1M tokens |
| `gemini-3-flash-preview` | 快速任务 | Large |
| `gemini-2.5-flash` | 旧版兼容 | Large |
| `gemini-2.5-flash-lite` | 最快，简单任务 | Medium |

```bash
# 首选 Pro 模型（使用 alias）
pgemini-pro -p "复杂分析" -o text

# 快速模型（使用 alias）
pgemini-flash -p "简单任务" -o text

# Pro 失败自动切换 Flash（正确写法：重复传递输入）
cat file.js | pgemini-pro -p "审查" || cat file.js | pgemini-flash -p "审查"
```

### 会话管理

| 参数 | 短写 | 描述 |
|------|------|------|
| `--resume` | `-r` | 按索引或 "latest" 恢复会话 |
| `--list-sessions` | | 列出可用会话 |
| `--delete-session` | | 按索引删除会话 |

### 执行选项

| 参数 | 短写 | 描述 |
|------|------|------|
| `--sandbox` | `-s` | 在隔离沙箱运行 |
| `--approval-mode` | | `default`, `auto_edit`, 或 `yolo` |
| `--timeout` | | 请求超时（毫秒） |
| `--checkpointing` | | 启用文件变更快照 |

### 上下文与工具

| 参数 | 描述 |
|------|------|
| `--include-directories` | 添加目录到工作区 |
| `--allowed-tools` | 限制可用工具 |
| `--allowed-mcp-server-names` | 限制 MCP 服务器 |

### 其他选项

| 参数 | 短写 | 描述 |
|------|------|------|
| `--debug` | `-d` | 启用调试输出 |
| `--version` | `-v` | 显示版本 |
| `--help` | `-h` | 显示帮助 |
| `--list-extensions` | `-l` | 列出已安装扩展 |
| `--prompt-interactive` | `-i` | 交互模式带初始提示 |

## 输出格式

### Text (`-o text`)

```bash
gemini "prompt" -o text
# 返回：人类可读响应
```

### JSON (`-o json`)

```bash
gemini "prompt" -o json
```

返回结构化数据：
```json
{
  "response": "实际响应内容",
  "stats": {
    "models": {
      "gemini-3.1-pro-preview": {
        "api": {
          "totalRequests": 3,
          "totalErrors": 0,
          "totalLatencyMs": 5000
        },
        "tokens": {
          "prompt": 1500,
          "candidates": 500,
          "total": 2000,
          "cached": 800,
          "thoughts": 150,
          "tool": 50
        }
      }
    },
    "tools": {
      "totalCalls": 2,
      "totalSuccess": 2,
      "totalFail": 0,
      "byName": {
        "google_web_search": {
          "count": 1,
          "success": 1,
          "durationMs": 3000
        }
      }
    }
  }
}
```

### Stream JSON (`-o stream-json`)

实时换行分隔 JSON 事件，用于监控长任务。

## 配置文件

### 设置位置

优先级顺序（从高到低）：
1. `/etc/gemini-cli/settings.json` (系统)
2. `~/.gemini/settings.json` (用户)
3. `.gemini/settings.json` (项目)

### 示例设置

```json
{
  "security": {
    "auth": {
      "selectedType": "oauth-personal"
    }
  },
  "general": {
    "previewFeatures": true,
    "vimMode": false,
    "checkpointing": true
  },
  "mcpServers": {}
}
```

### 项目上下文 (gemini.md)

在项目根目录创建 `.gemini/gemini.md`（全小写，跨平台兼容）：
```markdown
# 项目上下文

项目描述和指南。

## 编码标准
- Gemini 应遵循的标准

## 修改时
- 修改指南
```

### 忽略文件 (.geminiignore)

类似 `.gitignore`，排除文件：
```
node_modules/
dist/
*.log
.env
```

## 会话管理

### 列出会话

```bash
gemini --list-sessions
```

输出：
```
Available sessions for this project (5):
  1. Create task manager (10 minutes ago) [uuid]
  2. Review code (20 minutes ago) [uuid]
  ...
```

### 恢复会话

```bash
# 按索引
echo "follow-up question" | gemini -r 1 -o text

# 最新会话
echo "continue" | gemini -r latest -o text
```

## 速率限制

### 免费层限制

- 60 请求/分钟
- 1000 请求/天

### 速率限制行为

- CLI 自动重试，指数回退
- 消息：`"quota will reset after Xs"`
- 典型等待：1-5秒

### 缓解策略

1. 使用 `gemini-3-flash-preview` 处理简单任务
2. 批量操作合并到单个提示词
3. 长任务后台运行

## 交互命令

交互模式下可用斜杠命令：

| 命令 | 用途 |
|------|------|
| `/help` | 显示可用命令 |
| `/tools` | 列出可用工具 |
| `/stats` | 显示 token 使用 |
| `/compress` | 总结上下文节省 token |
| `/restore` | 恢复文件快照 |
| `/chat save <tag>` | 保存对话 |
| `/chat resume <tag>` | 恢复对话 |
| `/memory show` | 显示 GEMINI.md 上下文 |
| `/memory refresh` | 重载上下文文件 |

## 管道与脚本

### 管道输入

```bash
echo "What is 2+2?" | gemini -o text
cat file.txt | gemini "summarize this" -o text
```

### 文件引用语法

提示词中用 `@` 引用文件：
```bash
gemini "Review @./src/main.js for bugs" -o text
```

### Shell 命令执行

交互模式，前缀 `!`：
```
> !git status
```

## 键盘快捷键（交互模式）

| 快捷键 | 功能 |
|--------|------|
| `Ctrl+L` | 清屏 |
| `Ctrl+V` | 从剪贴板粘贴 |
| `Ctrl+Y` | 切换 YOLO 模式 |
| `Ctrl+X` | 打开外部编辑器 |

## 故障排除

### 常见问题

| 问题 | 解决方案 |
|------|----------|
| "API key not found" | 设置 `GEMINI_API_KEY` 环境变量 |
| "Rate limit exceeded" | 等待自动重试或使用 Flash |
| "Context too large" | 使用 `.geminiignore` 或精确指定 |
| "Tool call failed" | 检查 JSON 统计获取详情 |
| `network timeout` | 网络超时 | 检查代理设置或网络连接 |

### 调试模式

```bash
gemini "prompt" --debug -o text
```

### 错误报告

完整错误报告保存到：
```
/var/folders/.../gemini-client-error-*.json
```