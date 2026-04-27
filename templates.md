# Gemini CLI Prompt Templates (升级版)

可复用的提示词模板，已更新模型版本并改进代码审查流程。

> **前提**：以下示例假设已配置 `pgemini` alias。如未配置，请自行添加代理变量：
> ```bash
> https_proxy=http://127.0.0.1:7890 http_proxy=http://127.0.0.1:7890 gemini ...
> ```
> alias 配置方法见 SKILL.md。

## 代码生成

### 单文件应用

```bash
pgemini -p "创建一个 [描述] 包含 [功能]。包括 [需求]。输出完整文件内容。" --yolo -o text
```

**示例：**
```bash
pgemini -p "创建一个单文件 HTML/CSS/JS 计算器：基本运算、历史显示、键盘支持、暗色模式切换、响应式设计。输出完整文件内容。" --yolo -o text
```

### 多文件项目

```bash
pgemini -p "创建一个 [项目类型] 使用 [技术栈]。包含 [功能]。创建所有必要文件并使其可运行。使用现代最佳实践。立即开始构建。" --yolo -o text
```

**示例：**
```bash
pgemini -p "创建一个 REST API，使用 Express、SQLite 和 JWT 认证。包含用户 CRUD、输入验证、错误处理。创建所有必要文件并使其可运行。立即开始构建。" --yolo -o text
```

### 组件/模块

```bash
pgemini -p "创建一个 [组件类型] 功能是 [功能描述]。遵循 [标准]。包含 [需求]。输出代码。" --yolo -o text
```

**示例：**
```bash
pgemini -p "创建一个 React hook useLocalStorage，同步状态与 localStorage。遵循 React 18 最佳实践。包含 TypeScript 类型。输出代码。" --yolo -o text
```

## 代码审查（结构化流程）

### 综合审查模板

**重要**：fallback 时必须重新传递输入：

```bash
# 正确写法：重复传递 cat 输入
cat 文件路径 | pgeminip -p "
请作为资深软件工程师，审核以下代码（文件路径：[路径]，技术栈：[栈]）。

请重点从以下维度审查，按优先级输出：

1. 🚨 **高危问题**：安全漏洞、内存泄漏、崩溃风险
2. ⚠️ **潜在缺陷**：边界情况、竞态条件、错误处理
3. ⚡ **性能与可维持性**：复杂度、代码结构、命名规范

保持精简客观。
注意：请输出精简的修改建议（如 Patch 格式或只展示改动的前后几行），严禁重写未修改的大段原始代码。
" || cat 文件路径 | pgeminif -p "同上提示词"
```

### 安全专项审查

```bash
cat 文件路径 | pgeminip -p "
审查此代码的安全漏洞，包括：
- XSS（跨站脚本攻击）
- SQL 注入
- 命令注入
- 不安全数据处理
- 认证问题
报告发现并标注严重级别。
注意：请输出精简的修改建议，严禁重写未修改的大段原始代码。
"
```

### 性能审查

```bash
cat 文件路径 | pgeminip -p "
分析此代码的性能问题：
- 低效算法
- 内存泄漏
- 不必要重渲染
- 阻塞操作
- 优化机会
提供具体建议。保持精简。
"
```

### Git Diff 审查

```bash
git diff | pgeminip -p "
审查这些 Git 变更：
1. 功能完整性
2. 潜在 bug
3. 安全风险
4. 代码质量

保持精简，列出关键问题。
" || git diff | pgeminif -p "同上提示词"
```

## Bug 修复

### 修复已知 Bug

```bash
pgemini -p "修复 [文件] 中的这些 bug：
1) [Bug 描述]
2) [Bug 描述]
3) [Bug 描述]
立即应用修复。" --yolo -o text
```

### 自动检测并修复

```bash
pgemini -p "分析 [文件] 的 bug，然后修复所有发现的问题。立即应用修复。" --yolo -o text
```

## 测试生成

### 单元测试

```bash
pgemini -p "为 [文件] 生成 [框架] 单元测试。覆盖：
- 所有公开函数
- 边界情况
- 错误处理
- [特定区域]
输出完整测试文件。" --yolo -o text
```

**示例：**
```bash
pgemini -p "为 utils.js 生成 Jest 单元测试。覆盖：
- 所有导出函数
- 边界情况（空输入、null、undefined）
- 错误处理
- 边界条件
输出完整测试文件。" --yolo -o text
```

### 集成测试

```bash
pgemini -p "为 [组件/API] 生成集成测试。测试：
- 正常路径场景
- 错误场景
- 边界情况
使用 [框架]。输出完整测试文件。" --yolo -o text
```

## 文档生成

### JSDoc/TSDoc

```bash
pgemini -p "为 [文件] 所有函数生成 [JSDoc/TSDoc] 文档。包含：
- 函数描述
- 参数类型和描述
- 返回类型和描述
- 使用示例
输出 [格式]。" --yolo -o text
```

### README 生成

```bash
pgemini -p "为此项目生成 README.md。包含：
- 项目描述
- 安装说明
- 使用示例
- API 参考
- 贡献指南
使用代码库收集准确信息。" --yolo -o text
```

### API 文档

```bash
pgemini -p "记录 [文件/目录] 所有 API 端点。包含：
- HTTP 方法和路径
- 请求参数
- 请求体 schema
- 响应 schema
- 示例请求/响应
输出 [Markdown/OpenAPI] 格式。" --yolo -o text
```

## 代码转换

### 重构

```bash
pgemini -p "重构 [文件] 以：
- [具体改进]
- [具体改进]
保持所有现有功能。立即应用变更。" --yolo -o text
```

### 语言翻译

```bash
pgemini -p "将 [文件] 从 [源语言] 翻译到 [目标语言]。保持：
- 相同功能
- 类似代码结构
- 目标语言惯用模式
输出翻译后代码。" --yolo -o text
```

### 框架迁移

```bash
pgemini -p "将 [文件] 从 [旧框架] 转换到 [新框架]。保持所有功能。使用 [新框架] 最佳实践。输出转换后代码。" --yolo -o text
```

## 网络研究

### 当前信息

```bash
pgemini -p "截至 [日期]，[主题] 最新情况是什么？使用 Google Search 搜索当前信息。总结关键点。" -o text
```

### 库/API 研究

```bash
pgemini -p "研究 [库/API] 并提供：
- 最新版本和变更
- 最佳实践
- 常用模式
- 需要避免的问题
使用 Google Search 获取当前信息。" -o text
```

### 比较研究

```bash
pgemini -p "针对 [用例] 比较 [选项A] vs [选项B]。使用 Google Search 获取当前基准测试和社区意见。提供建议。" -o text
```

## 架构分析

### 项目分析

```bash
pgemini -p "使用 codebase_investigator 工具分析此项目。报告：
- 整体架构
- 关键依赖
- 组件关系
- 潜在问题" -o text
```

### 依赖分析

```bash
pgemini -p "分析此项目依赖：
- 直接 vs 传递依赖
- 过时包
- 安全漏洞
- 包大小影响
使用可用工具收集信息。" -o text
```

## 专门任务

### Git 提交消息

```bash
pgemini -p "分析暂存变更并生成遵循 conventional commits 格式的提交消息。简洁但有描述性。" -o text
```

### 代码解释

```bash
pgemini -p "详细解释 [文件/函数] 做什么：
- 目的和用例
- 逐步工作原理
- 关键算法/模式
- 依赖和副作用" -o text
```

### 错误诊断

```bash
pgemini -p "诊断此错误：
[错误消息]
上下文：[相关上下文]
提供：
- 根因
- 解决步骤
- 预防提示" -o text
```

## 模板变量

在模板中使用这些占位符：

- `[文件]` - 文件路径或名称
- `[目录]` - 目录路径
- `[描述]` - 简短描述
- `[功能]` - 功能列表
- `[需求]` - 具体需求
- `[框架]` - 测试/UI框架
- `[语言]` - 编程语言
- `[格式]` - 输出格式（markdown、JSON等）
- `[日期]` - 时间敏感查询的日期
- `[主题]` - 研究主题
- `[栈]` - 技术栈