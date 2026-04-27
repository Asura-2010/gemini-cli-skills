# Gemini CLI Integration Patterns (升级版)

高级 Gemini CLI 集成模式，已更新模型版本。

## Pattern 1: Generate-Review-Fix 循环

最可靠的代码生成质量保证模式。

```bash
# Step 1: 生成代码
https_proxy=http://127.0.0.1:7890 gemini "创建 [代码描述]" --yolo -o text

# Step 2: 让 Gemini 审查自己的工作
https_proxy=http://127.0.0.1:7890 gemini "审查 [生成的文件] 的 bug 和安全问题" -o text

# Step 3: 修复发现的问题
https_proxy=http://127.0.0.1:7890 gemini "修复 [文件] 中的这些问题：[审查列出的问题]。立即应用。" --yolo -o text
```

### 为什么有效

- 生成 vs 审查使用不同的"思维模式"
- 自我纠正捕获常见错误
- 安全漏洞通常在审查阶段发现

### 示例

```bash
# 生成
https_proxy=http://127.0.0.1:7890 gemini "创建用户认证模块，使用 bcrypt 和 JWT" --yolo -o text

# 审查
https_proxy=http://127.0.0.1:7890 gemini "审查 auth.js 的安全漏洞" -o text
# 输出："发现 XSS 风险、缺少输入验证、JWT 密钥过弱"

# 修复
https_proxy=http://127.0.0.1:7890 gemini "修复 auth.js：XSS 风险、添加输入验证、使用环境变量存储 JWT 密钥。立即应用。" --yolo -o text
```

## Pattern 2: JSON 输出用于程序化处理

需要程序化处理结果时使用 JSON 输出。

```bash
https_proxy=http://127.0.0.1:7890 gemini "[提示词]" -o json 2>&1
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

长任务后台执行，继续其他工作。

```bash
# 后台启动
https_proxy=http://127.0.0.1:7890 gemini "[长任务]" --yolo -o text 2>&1 &

# 获取进程 ID 用于后续监控
echo $!

# 使用 BashOutput 工具增量监控
```

### 适用场景

- 大型项目代码生成
- 文档生成
- 同时运行多个 Gemini 任务

### 并行执行

```bash
# 同时运行多个任务
https_proxy=http://127.0.0.1:7890 gemini "创建前端" --yolo -o text 2>&1 &
https_proxy=http://127.0.0.1:7890 gemini "创建后端" --yolo -o text 2>&1 &
https_proxy=http://127.0.0.1:7890 gemini "创建测试" --yolo -o text 2>&1 &
```

## Pattern 4: 模型选择策略

为任务选择合适的模型。

### 决策树

```
任务复杂吗（架构、多文件、深度分析）？
├── 是 → 使用 Pro (gemini-3.1-pro-preview)
└── 否 → 速度关键吗？
    ├── 是 → 使用 Flash (gemini-3-flash-preview)
    └── 否 → 任务琐碎吗（格式化、简单查询）？
        ├── 是 → 使用 Flash Lite (gemini-2.5-flash-lite)
        └── 否 → 使用 Flash (gemini-3-flash-preview)
```

### 示例

```bash
# 复杂：架构分析
https_proxy=http://127.0.0.1:7890 gemini -m gemini-3.1-pro-preview --skip-trust -p "分析代码库架构" -o text

# 快速：简单格式化
https_proxy=http://127.0.0.1:7890 gemini -m gemini-3-flash-preview --skip-trust -p "格式化此 JSON" -o text

# 琐碎：一句话
https_proxy=http://127.0.0.1:7890 gemini -m gemini-2.5-flash-lite --skip-trust -p "2+2等于几？" -o text
```

## Pattern 5: 速率限制处理

在速率限制内工作的策略。

### 方法1：让自动重试处理

默认行为 - CLI 自动重试带回退。

### 方法2：低优先级使用 Flash

```bash
# 高优先级：使用 Pro
https_proxy=http://127.0.0.1:7890 gemini -m gemini-3.1-pro-preview --skip-trust -p "[重要任务]" -o text

# 低优先级：使用 Flash（不同配额）
https_proxy=http://127.0.0.1:7890 gemini -m gemini-3-flash-preview --skip-trust -p "[不那么关键的任务]" -o text
```

### 方法3：批量操作

合并相关操作到单个提示词：
```bash
# 避免多次调用：
https_proxy=http://127.0.0.1:7890 gemini "创建文件 A" --yolo
https_proxy=http://127.0.0.1:7890 gemini "创建文件 B" --yolo
https_proxy=http://127.0.0.1:7890 gemini "创建文件 C" --yolo

# 单次调用：
https_proxy=http://127.0.0.1:7890 gemini "创建文件 A、B、C，包含 [规范]。现在创建所有文件。" --yolo
```

### 方法4：顺序带延迟

自动化脚本添加延迟：
```bash
https_proxy=http://127.0.0.1:7890 gemini "[任务 1]" --yolo -o text
sleep 2
https_proxy=http://127.0.0.1:7890 gemini "[任务 2]" --yolo -o text
```

## Pattern 6: 上下文丰富

提供丰富上下文获得更好结果。

### 使用文件引用

```bash
https_proxy=http://127.0.0.1:7890 gemini "基于 @./package.json 和 @./src/index.js，提供改进建议" -o text
```

### 使用 GEMINI.md

创建自动包含的项目上下文：
```markdown
# .gemini/GEMINI.md

## 项目概述
这是一个使用 TypeScript 的 React 应用。

## 编码标准
- 使用函数组件
- 优先 hooks 而非类
- 所有函数需要 JSDoc
```

### 提示词中显式上下文

```bash
https_proxy=http://127.0.0.1:7890 gemini "给定此上下文：
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
https_proxy=http://127.0.0.1:7890 gemini "创建工具函数" --yolo -o text

# 验证
node --check utils.js && eslint utils.js && npm test
```

## Pattern 8: 增量优化

分阶段构建复杂输出。

```bash
# Stage 1: 核心结构
https_proxy=http://127.0.0.1:7890 gemini "创建基本 Express 服务器，路由 /api/users" --yolo -o text

# Stage 2: 添加功能
https_proxy=http://127.0.0.1:7890 gemini "为 server.js 中 Express 服务器添加认证中间件" --yolo -o text

# Stage 3: 添加另一功能
https_proxy=http://127.0.0.1:7890 gemini "为 server.js 中 Express 服务器添加速率限制" --yolo -o text

# Stage 4: 审查全部
https_proxy=http://127.0.0.1:7890 gemini "审查 server.js 问题并优化" -o text
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
https_proxy=http://127.0.0.1:7890 gemini "审查此代码的 bug 和安全问题：[粘贴代码]" -o text
```

### Gemini 生成，Claude 审查

```bash
# 1. Gemini 生成
https_proxy=http://127.0.0.1:7890 gemini "创建 [代码]" --yolo -o text

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
https_proxy=http://127.0.0.1:7890 gemini "分析此代码库架构" -o text
# 会话自动保存

# 列出会话
gemini --list-sessions

# 继续跟进
echo "你发现了什么模式？" | https_proxy=http://127.0.0.1:7890 gemini -r 1 -o text

# 进一步优化
echo "聚焦认证流程" | https_proxy=http://127.0.0.1:7890 gemini -r 1 -o text
```

### 适用场景

- 迭代分析
- 建立在前文基础上
- 调试会话

## Pattern 11: Pro-Flash 自动切换

Pro 失败自动切换 Flash。

```bash
# 管道方式（推荐）
cat file.js | https_proxy=http://127.0.0.1:7890 \
gemini -m gemini-3.1-pro-preview --skip-trust -p "审查此代码" || \
gemini -m gemini-3-flash-preview --skip-trust -p "审查此代码"
```

### 适用场景

- Pro 配额用完时
- Pro 速率限制触发时
- 确保任务完成不中断

## 反模式：避免

### 不要：期望立即执行

YOLO 模式不阻止规划。Gemini 可能仍展示计划。

**正确做法**：使用强制语言（"现在应用"、"立即开始"）

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