# Gemini CLI Integration Patterns

高级 Gemini CLI 集成模式，已更新模型版本。

> **前提**：以下示例假设已配置 `pgemini`、`pgemini-pro`、`pgemini-flash` alias。如未配置，请自行添加代理变量。alias 配置方法见 SKILL.md。

## Pattern 1: Generate-Review-Fix 循环

最可靠的代码生成质量保证模式。

```bash
# Step 1: 生成代码
pgemini -p "创建 [代码描述]" --yolo -o text

# Step 2: 让 Gemini 审查自己的工作
pgemini -p "审查 [生成的文件] 的 bug 和安全问题" -o text

# Step 3: 修复发现的问题
pgemini -p "修复 [文件] 中的这些问题：[审查列出的问题]。立即应用。" --yolo -o text
```

### 为什么有效

- 生成 vs 审查使用不同的"思维模式"
- 自我纠正捕获常见错误
- 安全漏洞通常在审查阶段发现

### 示例

```bash
# 生成
pgemini -p "创建用户认证模块，使用 bcrypt 和 JWT" --yolo -o text

# 审查
pgemini -p "审查 auth.js 的安全漏洞" -o text
# 输出："发现 XSS 风险、缺少输入验证、JWT 密钥过弱"

# 修复
pgemini -p "修复 auth.js：XSS 风险、添加输入验证、使用环境变量存储 JWT 密钥。立即应用。" --yolo -o text
```

## Pattern 2: JSON 输出用于程序化处理

需要程序化处理结果时使用 JSON 输出。

```bash
pgemini -p "[提示词]" -o json 2>&1
```

### 解析响应

```javascript
// Node.js 或使用 jq
const result = JSON.parse(output);
const content = result.response;
const tokenUsage = result.stats.models["gemini-3.1-pro-preview"].tokens.total;
const toolCalls = result.stats.tools.byName;
```

### 用例

- 从响应提取特定数据
- 监控 token 使用
- 跟踪工具调用成功/失败
- 构建自动化管道

## Pattern 3: 后台执行

长任务后台执行，输出重定向到文件避免干扰终端。

```bash
# 输出重定向到日志文件
pgemini -p "[长任务]" --yolo -o text > gemini_output.log 2>&1 &

# 监控进度
tail -f gemini_output.log

# 查看是否完成
ps aux | grep gemini
```

### 适用场景

- 大型项目代码生成
- 文档生成
- 同时运行多个 Gemini 任务

### 并行执行

```bash
# 同时运行多个任务（每个输出到不同文件）
pgemini -p "创建前端" --yolo -o text > frontend.log 2>&1 &
pgemini -p "创建后端" --yolo -o text > backend.log 2>&1 &
pgemini -p "创建测试" --yolo -o text > tests.log 2>&1 &
```

## Pattern 4: 模型选择策略

为任务选择合适的模型。

### 决策树

```
任务复杂吗（架构、多文件、深度分析）？
├── 是 → 使用 Pro (gemini-3.1-pro-preview) → pgemini-pro
└── 否 → 速度关键吗？
    ├── 是 → 使用 Flash (gemini-3-flash-preview) → pgemini-flash
    └── 否 → 任务琐碎吗（格式化、简单查询）？
        ├── 是 → 使用 Flash Lite (gemini-2.5-flash-lite)
        └── 否 → 使用 Flash (gemini-3-flash-preview) → pgemini-flash
```

### 示例

```bash
# 复杂：架构分析
pgemini-pro -p "分析代码库架构" -o text

# 快速：简单格式化
pgemini-flash -p "格式化此 JSON" -o text

# 琐碎：一句话
pgemini -m gemini-2.5-flash-lite -p "2+2等于几？" -o text
```

## Pattern 5: 速率限制处理

在速率限制内工作的策略。

### 方法1：让自动重试处理

默认行为 - CLI 自动重试带回退。

### 方法2：低优先级使用 Flash

```bash
# 高优先级：使用 Pro
pgemini-pro -p "[重要任务]" -o text

# 低优先级：使用 Flash（不同配额）
pgemini-flash -p "[不那么关键的任务]" -o text
```

### 方法3：批量操作

合并相关操作到单个提示词：
```bash
# 避免多次调用：
pgemini -p "创建文件 A" --yolo -o text
pgemini -p "创建文件 B" --yolo -o text
pgemini -p "创建文件 C" --yolo -o text

# 单次调用：
pgemini -p "创建文件 A、B、C，包含 [规范]。现在创建所有文件。" --yolo -o text
```

### 方法4：顺序带延迟

自动化脚本添加延迟：
```bash
pgemini -p "[任务 1]" --yolo -o text
sleep 2
pgemini -p "[任务 2]" --yolo -o text
```

## Pattern 6: 上下文丰富

提供丰富上下文获得更好结果。

### 使用文件引用

```bash
pgemini -p "基于 @./package.json 和 @./src/index.js，提供改进建议" -o text
```

### 使用 gemini.md

创建自动包含的项目上下文：
```markdown
# .gemini/gemini.md（全小写）

## 项目概述
这是一个使用 TypeScript 的 React 应用。

## 编码标准
- 使用函数组件
- 优先 hooks 而非类
- 所有函数需要 JSDoc
```

### 提示词中显式上下文

```bash
pgemini -p "给定此上下文：
- 项目使用 React 18 和 TypeScript
- 状态管理：Zustand
- 样式：Tailwind CSS

创建用户 Profile 组件。" --yolo -o text
```

## Pattern 7: 验证管道

使用 Gemini 输出前始终验证。

### 验证步骤

1. **语法检查**
   ```bash
   # JavaScript
   node --check generated.js

   # TypeScript
   tsc --noEmit generated.ts
   ```

2. **安全扫描**
   - 检查用户输入的 innerHTML（XSS）
   - 查找 eval() 或 Function() 调用
   - 验证输入验证

3. **功能测试**
   - 运行生成的测试
   - 手动冒烟测试

4. **风格检查**
   ```bash
   eslint generated.js
   prettier --check generated.js
   ```

### 自动验证模式

```bash
# 生成
pgemini -p "创建工具函数" --yolo -o text

# 验证
node --check utils.js && eslint utils.js && npm test
```

## Pattern 8: 增量优化

分阶段构建复杂输出。

```bash
# Stage 1: 核心结构
pgemini -p "创建基本 Express 服务器，路由 /api/users" --yolo -o text

# Stage 2: 添加功能
pgemini -p "为 server.js 中 Express 服务器添加认证中间件" --yolo -o text

# Stage 3: 添加另一功能
pgemini -p "为 server.js 中 Express 服务器添加速率限制" --yolo -o text

# Stage 4: 审查全部
pgemini -p "审查 server.js 问题并优化" -o text
```

### 优势

- 更容易调试问题
- 每阶段验证后再继续
- 清晰审计跟踪

## Pattern 9: Claude-Gemini 交叉验证

两个 AI 获得最高质量。

### Claude 生成，Gemini 审查

```bash
# 1. Claude 写代码（使用 Claude Code 正常工具）
# 2. Gemini 审查
cat generated.js | pgemini-pro -p "审查此代码的 bug 和安全问题" || \
cat generated.js | pgemini-flash -p "审查此代码的 bug 和安全问题"
```

### Gemini 生成，Claude 审查

```bash
# 1. Gemini 生成
pgemini -p "创建 [代码]" --yolo -o text

# 2. Claude 审查输出（在对话中）
# "审查 Gemini 生成的这段代码..."
```

### 不同视角

- Claude：推理强、遵循复杂指令
- Gemini：实时网络知识、代码库调查强

## Pattern 10: 会话连续性

多轮工作流使用会话。

```bash
# 初始任务
pgemini -p "分析此代码库架构" -o text
# 会话自动保存

# 列出会话
gemini --list-sessions

# 继续跟进
echo "你发现了什么模式？" | pgemini -r 1 -o text

# 进一步优化
echo "聚焦认证流程" | pgemini -r 1 -o text
```

### 适用场景

- 迭代分析
- 建立在前文基础上
- 调试会话

## Pattern 11: Pro-Flash 正确切换

**重要**：管道输入时，fallback 必须重新传递输入，否则备用模型无法获取内容。

```bash
# 正确写法：重复传递输入
cat file.js | pgemini-pro -p "审查此代码" || \
cat file.js | pgemini-flash -p "审查此代码"

# 或使用变量存储
CONTENT=$(cat file.js)
echo "$CONTENT" | pgemini-pro -p "审查此代码" || \
echo "$CONTENT" | pgemini-flash -p "审查此代码"
```

### 适用场景

- Pro 配额用完时
- Pro 速率限制触发时
- 确保任务完成不中断

## 反模式：避免

### 不要：管道 fallback 不重新传递输入

第二个命令无法获取文件内容，会死等或执行失败。

**正确做法**：fallback 时重新 `cat` 文件

### 不要：忽略速率限制

频繁请求浪费时间在重试。

**正确做法**：使用合适模型、批量操作

### 不要：盲目信任输出

Gemini 可能犯错，尤其安全方面。

**正确做法**：始终验证生成的代码

### 不要：单个提示词过度指定

极长提示词可能混淆模型。

**正确做法**：复杂任务使用增量优化

### 不要：忘记上下文限制

即使 1M tokens，上下文也可能溢出。

**正确做法**：使用 .geminiignore、精确指定文件