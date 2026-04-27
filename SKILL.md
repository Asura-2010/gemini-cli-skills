---
name: gemini-cli
description: Wield Google's Gemini CLI as a powerful auxiliary tool for code generation, review, analysis, and web research. Use when tasks benefit from a second AI perspective, current web information via Google Search, codebase architecture analysis, or parallel code generation. Also use when user explicitly requests Gemini operations. 国内环境需配置代理。
allowed-tools:
  - Bash
  - Read
  - Write
  - Grep
  - Glob
---

# Gemini CLI Integration Skill (升级版)

整合 Gemini CLI (v0.16.0+) 与 Gemini 3.x 系列，专为国内环境优化。

## 前提条件

- Gemini CLI 已安装并可用 (`gemini` 命令)
- 已通过 Google 账户认证
- **网络要求**：国内需开启系统代理访问 Google API

## 代理配置（国内环境必读）

Gemini CLI 需要访问 Google API，在国内环境必须使用代理。

### 检查代理状态

```bash
echo "https_proxy=$https_proxy http_proxy=$http_proxy"
```

### 推荐：配置 Shell Alias

为避免每个命令都手写代理变量，建议配置 alias：

```bash
# 添加到 ~/.zshrc 或 ~/.bashrc
alias pgemini='https_proxy=http://127.0.0.1:7890 http_proxy=http://127.0.0.1:7890 gemini'
alias pgeminif='https_proxy=http://127.0.0.1:7890 http_proxy=http://127.0.0.1:7890 gemini -m gemini-3-flash-preview --skip-trust'
alias pgeminip='https_proxy=http://127.0.0.1:7890 http_proxy=http://127.0.0.1:7890 gemini -m gemini-3.1-pro-preview --skip-trust'

# 后续命令简化为：
pgeminip -p "提示词"
```

### 单命令代理设置（临时）

如未配置 alias，可单命令设置代理：

```bash
https_proxy=http://127.0.0.1:7890 http_proxy=http://127.0.0.1:7890 \
gemini -m gemini-3.1-pro-preview --skip-trust --yolo -p "提示词"
```

### 代理地址参考

| 代理软件 | 默认端口 |
|---------|---------|
| Clash | 7890 |
| V2RayN | 10809 |
| Shadowsocks | 1080 |

> 根据实际代理软件调整端口

## 模型版本（已更新）

| 优先级 | 模型 | 说明 |
|-------|------|------|
| 首选 | `gemini-3.1-pro-preview` | 最强推理能力 |
| 备用 | `gemini-3-flash-preview` | 速度快，Pro 额度用完时自动切换 |
| 旧版 | `gemini-2.5-flash` | 兼容旧版本 CLI |

## 使用时机

### 适用场景

1. **第二意见 / 交叉验证**
   - 代码审查（不同 AI 视角）
   - 安全审计
   - 发现 Claude 可能遗漏的问题

2. **Google Search 实时搜索**
   - 需要当前网络信息的问题
   - 最新库版本、API 变化、文档更新
   - 当前事件或最新发布

3. **代码库架构分析**
   - 使用 Gemini 的 `codebase_investigator` 工具
   - 理解陌生代码库
   - 映射跨文件依赖关系

4. **并行处理**
   - 后台执行任务时继续其他工作
   - 同时运行多个代码生成
   - 后台文档生成

5. **专门生成**
   - 测试套件生成
   - JSDoc/文档生成
   - 代码语言翻译

### 不适用场景

- 简单快速任务（开销不值得）
- 需要即时响应的任务（速率限制导致延迟）
- 上下文已加载和理解的情况
- 需要对话式交互优化的任务

## 核心指令

### 1. 验证安装

```bash
command -v gemini || which gemini
```

### 2. 基本命令模式

```bash
# 国内环境带代理（使用 alias 后）
pgemini -p "[prompt]" --yolo -o text

# 或完整写法
https_proxy=http://127.0.0.1:7890 http_proxy=http://127.0.0.1:7890 \
gemini -p "[prompt]" --yolo -o text 2>&1
```

关键参数：
- `--yolo` 或 `-y`: 自动批准所有工具调用
- `--skip-trust`: 跳过信任检查
- `-o text`: 人类可读输出
- `-o json`: 结构化输出带统计
- `-m gemini-3-flash-preview`: 简单任务用快速模型

### 3. 关键行为注意

**YOLO 模式行为**：自动批准工具调用但不会阻止规划提示。Gemini 可能仍会展示计划并询问"这个计划好吗？"。使用强制语言：
- "现在应用"
- "立即开始"
- "不做确认直接执行"

**速率限制**：免费层 60请求/分钟，1000/天。CLI 自动重试带回退。预期看到"quota will reset after Xs"消息。

### 4. 输出处理

JSON 输出 (`-o json`) 解析：
```json
{
  "response": "实际内容",
  "stats": {
    "models": { "tokens": {...} },
    "tools": { "byName": {...} }
  }
}
```

## 快速参考命令

以下示例假设已配置 `pgemini` alias，或请自行添加代理变量。

### 代码生成

```bash
pgemini -p "创建 [描述] 包含 [功能]。输出完整文件内容。" --yolo -o text
```

### 代码审查（管道方式，带正确 fallback）

**重要**：fallback 时必须重新传递输入，否则备用模型无法获取文件内容：

```bash
# 正确写法：重复传递 cat 输入
cat 文件路径 | pgeminip -p "审查提示词" || \
cat 文件路径 | pgeminif -p "审查提示词"

# 或使用变量存储文件内容
FILE_CONTENT=$(cat 文件路径)
echo "$FILE_CONTENT" | pgeminip -p "审查提示词" || \
echo "$FILE_CONTENT" | pgeminif -p "审查提示词"
```

### Bug修复

```bash
pgemini -p "修复 [文件] 中的这些bug：[列表]。立即应用修复。" --yolo -o text
```

### 测试生成

```bash
pgemini -p "为 [文件] 生成 [Jest/pytest] 测试。重点关注 [区域]。" --yolo -o text
```

### 文档生成

```bash
pgemini -p "为 [文件] 所有函数生成 JSDoc。输出 markdown 格式。" --yolo -o text
```

### 架构分析

```bash
pgemini -p "使用 codebase_investigator 分析此项目" -o text
```

### 网络研究

```bash
pgemini -p "[主题] 最新情况是什么？使用 Google Search。" -o text
```

## 结构化代码审查流程

### 1. 确认审查范围

- 单文件：直接审查
- Git变更：使用 `git diff`
- 多文件/目录：合并后审查

### 2. 过滤无关文件（重要！）

排除以下文件：
- `package-lock.json`
- `*.min.js`、`*.min.css`
- `.svg` 图片文件
- `node_modules/` 目录
- 自动生成的文件、配置文件（除非用户明确要求）

### 3. 处理超长文件

超过 3000 行的文件：
- 询问用户是否只审查特定部分
- 或使用 `head -n 截取行数` 限制

### 4. 结构化提示词模板

```
请作为资深软件工程师，审核以下代码（文件路径：[路径]，技术栈：[栈]）。

请重点从以下维度审查，按优先级输出：

1. 🚨 **高危问题**：安全漏洞、内存泄漏、崩溃风险
2. ⚠️ **潜在缺陷**：边界情况、竞态条件、错误处理
3. ⚡ **性能与可维持性**：复杂度、代码结构、命名规范

保持精简客观。
注意：请输出精简的修改建议（如 Patch 格式或只展示改动的前后几行），严禁重写未修改的大段原始代码。
```

### 5. 执行审查（带正确 fallback）

```bash
# 正确写法：fallback 时重新传递输入
cat 文件路径 | pgeminip -p "审查提示词" || \
cat 文件路径 | pgeminif -p "审查提示词"
```

## 错误处理

### 速率限制超出

- CLI 自动重试带回退
- 使用 `-m gemini-3-flash-preview` 处理低优先级任务
- 长操作后台运行

### 命令失败

- 检查 JSON 输出获取详细错误统计
- 验证 Gemini 认证：`gemini --version`
- 检查 `~/.gemini/settings.json` 配置问题
- 确认代理设置正确

### 生成后验证

始终验证 Gemini 输出：
- 检查安全漏洞（XSS、注入）
- 测试功能符合需求
- 审查代码风格一致性
- 验证依赖适当性

## 集成工作流

### 标准 Generate-Review-Fix 循环

```bash
# 1. 生成
pgemini -p "创建 [代码]" --yolo -o text

# 2. 审查（Gemini 审查自己的工作）
pgemini -p "审查 [文件] 的 bug 和安全问题" -o text

# 3. 修复发现的问题
pgemini -p "修复 [文件] 中的 [问题]。立即应用。" --yolo -o text
```

### 后台执行

长任务后台运行，输出重定向到文件避免干扰终端：

```bash
# 输出重定向到日志文件
pgemini -p "[长任务]" --yolo -o text > gemini_output.log 2>&1 &

# 监控进度
tail -f gemini_output.log

# 查看是否完成
ps aux | grep gemini
```

## Gemini 独特能力

这些工具仅通过 Gemini 可用：

1. **google_web_search** - Google 实时互联网搜索
2. **codebase_investigator** - 深度架构分析
3. **save_memory** - 跨会话持久记忆

## 配置

### 项目上下文（可选）

在项目根目录创建 `.gemini/gemini.md`（全小写，跨平台兼容）作为持久上下文，Gemini 会自动读取。

### 会话管理

列出会话：`gemini --list-sessions`
恢复会话：`echo "follow-up" | gemini -r [index] -o text`

## 参考文档

- `reference.md` - 完整命令和参数参考
- `templates.md` - 常用操作的提示词模板
- `patterns.md` - 高级集成模式
- `tools.md` - Gemini 内置工具文档