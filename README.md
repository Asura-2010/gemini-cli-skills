# Gemini CLI Skill for Claude Code

整合 Google Gemini CLI 作为 Claude Code 的强力辅助工具，专为需要代理的网络环境优化。

> 基于 [forayconsulting/gemini_cli_skill](https://github.com/forayconsulting/gemini_cli_skill) 扩展，感谢原作者的贡献。

## 增强特性

本版本在原版基础上增强了：
- **模型版本更新**：支持 Gemini 3.x 系列
- **代理环境适配**：详细的代理配置指南，适配需要代理访问 Google API 的网络环境
- **结构化代码审查**：专业审查流程和规则
- **Shell Alias 配置**：简化命令输入
- **常见错误码说明**：快速定位问题

## 功能概述

- **代码生成** - 创建应用、组件、模块
- **代码审查** - 安全审计、缺陷检测、改进建议
- **测试生成** - 单元测试、集成测试
- **文档生成** - JSDoc、README、API文档
- **网络研究** - 通过 Google Search 获取实时信息
- **架构分析** - codebase_investigator 深度分析

## 安装

```bash
# 克隆仓库
git clone https://github.com/Asura-2010/gemini-cli-skills.git

# 复制到 Claude Code skills 目录
cp -r gemini-cli-skills ~/.claude/skills/gemini-cli
```

或手动创建 `~/.claude/skills/gemini-cli/` 目录并复制文件。

## 前提条件

- [Gemini CLI](https://github.com/google-gemini/gemini-cli) 已安装
- Gemini API key 或 OAuth 认证已配置
- **需要代理的网络环境**：需配置代理访问 Google API

```bash
# 安装 Gemini CLI
npm install -g @google/gemini-cli

# 认证（首次运行会提示）
gemini
```

## 文件说明

| 文件 | 用途 |
|------|------|
| `SKILL.md` | 主技能定义 - 使用时机、核心指令 |
| `reference.md` | CLI 命令和参数完整参考 |
| `templates.md` | 可复用的提示词模板 |
| `patterns.md` | 集成模式和最佳实践 |
| `tools.md` | Gemini 内置工具文档 |

## 使用方式

安装后，Claude Code 会自动在合适时机使用此技能：

```
"用 Gemini 审查这段代码的安全问题"
"让 Gemini 生成这个模块的测试"
"让 Gemini 搜索 TypeScript 5.5 的新特性"
"用 Gemini 分析这个项目的架构"
```

## 模型版本

| 优先级 | 模型 | 说明 |
|-------|------|------|
| 首选 | `gemini-3.1-pro-preview` | 最强推理能力 |
| 备用 | `gemini-3-flash-preview` | 速度快，Pro 额度用完时切换 |

## 快速参考

**推荐**：先配置 alias（见下文），后续命令更简洁。

```bash
# 基础生成
pgemini -p "创建 [描述]" --yolo -o text

# 代码审查（管道方式，正确 fallback）
cat 文件路径 | pgemini-pro -p "审查提示词" || cat 文件路径 | pgemini-flash -p "审查提示词"

# 网络研究
pgemini -p "搜索 [主题] 最新信息" -o text

# 架构分析
pgemini -p "用 codebase_investigator 分析项目" -o text
```

### Shell Alias 配置

```bash
# 添加到 ~/.zshrc 或 ~/.bashrc
# 根据实际代理软件调整端口（示例为 Clash 默认端口 7890）
alias pgemini='https_proxy=http://127.0.0.1:7890 http_proxy=http://127.0.0.1:7890 gemini'
alias pgemini-pro='https_proxy=http://127.0.0.1:7890 http_proxy=http://127.0.0.1:7890 gemini -m gemini-3.1-pro-preview --skip-trust'
alias pgemini-flash='https_proxy=http://127.0.0.1:7890 http_proxy=http://127.0.0.1:7890 gemini -m gemini-3-flash-preview --skip-trust'
```

## 为什么在 Claude Code 中使用 Gemini？

| 用例 | 优势 |
|------|------|
| 第二意见 | 不同 AI 视角的代码审查 |
| 实时信息 | Google Search 实时搜索 |
| 架构分析 | codebase_investigator 工具 |
| 并行工作 | 后台执行，继续其他任务 |

## License

MIT