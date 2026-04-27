# Gemini CLI Built-in Tools (升级版)

Gemini CLI 内置工具参考，模型版本已更新。

> **前提**：以下示例假设已配置 `pgemini` alias。如未配置，请自行添加代理变量。alias 配置方法见 SKILL.md。

## 独特工具（Claude Code 中不可用）

这些工具仅通过 Gemini CLI 可用：

### google_web_search

使用 Google Search API 执行网络搜索。

**能力：**
- 实时互联网搜索
- 当前信息（新闻、发布、文档）
- 带来源的接地响应

**使用：**
```bash
pgemini -p "React 19 最新特性是什么？使用 Google Search。" -o text
```

**最适合：**
- 当前事件和新闻
- 最新库版本
- 最近文档更新
- 社区意见和基准测试

**示例查询：**
- "lodash 4.x 有哪些安全漏洞？使用 Google Search。"
- "TypeScript 5.4 有什么新特性？使用 Google Search。"
- "2025年11月 Next.js 14 app router 最佳实践。"

---

### codebase_investigator

深度代码库分析的专门工具。

**能力：**
- 架构映射
- 依赖分析
- 跨文件关系检测
- 系统级模式识别

**使用：**
```bash
pgemini -p "使用 codebase_investigator 工具分析此项目" -o text
```

**输出包含：**
- 整体架构描述
- 关键文件用途
- 组件关系
- 依赖链
- 潜在问题/不一致

**最适合：**
- 入门新代码库
- 理解遗留系统
- 发现隐藏依赖
- 架构文档

**示例：**
```bash
pgemini -p "使用 codebase_investigator 映射此项目的认证流程" -o text
```

---

### save_memory

保存信息到持久长期记忆。

**能力：**
- 跨会话持久
- 键值存储
- 未来会话调用

**使用：**
```bash
pgemini -p "记住此项目使用 Zustand 进行状态管理。保存到记忆。" -o text
```

**最适合：**
- 项目约定
- 用户偏好
- 循环上下文
- 自定义指令

---

## 标准工具

这些工具类似 Claude Code 的能力：

### list_directory

列出路径中的文件和子目录。

**参数：**
- `path`: 要列出的目录
- `ignore`: 要排除的 glob 模式

**示例输出：**
```
src/
  components/
  utils/
  index.js
package.json
README.md
```

---

### read_file

读取文件内容，大文件截断。

**支持格式：**
- 文本文件（所有类型）
- 图片（PNG, JPG, GIF, WEBP, SVG, BMP）
- PDF 文档

**参数：**
- `path`: 文件路径
- `offset`: 起始行（大文件）
- `limit`: 行数

**大文件处理：**
如果文件超出限制，输出指示截断并提供使用 offset/limit 读取更多的说明。

---

### search_file_content

ripgrep 驱动的快速内容搜索。

**优于 grep：**
- 优化性能
- 自动输出限制（最多 20k 匹配）
- 更好的模式匹配

**参数：**
- `pattern`: 正则模式
- `path`: 搜索根目录
- 各种 ripgrep 参数

---

### glob

模式基于的文件查找。

**返回：**
- 绝对路径
- 按修改时间排序（最新优先）

**示例模式：**
- `src/**/*.ts` - src 中所有 TypeScript 文件
- `**/*.test.js` - 所有测试文件
- `**/README.md` - 所有 README

---

### web_fetch

从 URL 获取内容。

**能力：**
- HTTP/HTTPS URL
- 本地地址（localhost）
- 每请求最多 20 URL

**使用：**
```bash
pgemini -p "获取并总结 https://example.com/docs" -o text
```

---

### write_todos

内部任务跟踪。

**能力：**
- 跟踪复杂请求的子任务
- 组织多步工作
- 防止遗漏步骤

**自动使用：**
Gemini 在内部用于复杂任务。

---

## 工具调用

### 自动工具选择

Gemini 根据提示词自动选择合适工具：

| 提示词类型 | 选择工具 |
|------------|----------|
| "src/ 有什么文件" | list_directory |
| "找所有 TODO 注释" | search_file_content |
| "读 package.json" | read_file |
| "找所有 React 组件" | glob |
| "Vue 4 有什么新东西？" | google_web_search |
| "分析此代码库" | codebase_investigator |

### 显式工具请求

可以显式请求工具：

```bash
pgemini -p "使用 codebase_investigator 工具..." -o text
pgemini -p "搜索网络..." -o text
pgemini -p "使用 glob 找所有..." -o text
```

---

## JSON 输出中的工具统计

使用 `-o json` 时报告工具使用：

```json
{
  "stats": {
    "tools": {
      "totalCalls": 3,
      "totalSuccess": 3,
      "totalFail": 0,
      "totalDurationMs": 5000,
      "totalDecisions": {
        "accept": 0,
        "reject": 0,
        "modify": 0,
        "auto_accept": 3
      },
      "byName": {
        "google_web_search": {
          "count": 1,
          "success": 1,
          "fail": 0,
          "durationMs": 3000,
          "decisions": {
            "auto_accept": 1
          }
        },
        "read_file": {
          "count": 2,
          "success": 2,
          "fail": 0,
          "durationMs": 2000,
          "decisions": {
            "auto_accept": 2
          }
        }
      }
    }
  }
}
```

---

## Claude Code 工具对比

| 能力 | Claude Code | Gemini CLI |
|------|-------------|------------|
| 文件列表 | LS, Glob | list_directory, glob |
| 文件读取 | Read | read_file |
| 文件写入 | Write, Edit | write_file (YOLO) |
| 代码搜索 | Grep | search_file_content |
| 网络获取 | WebFetch | web_fetch |
| 网络搜索 | WebSearch | **google_web_search** |
| 架构 | Task (Explore) | **codebase_investigator** |
| 记忆 | N/A | **save_memory** |
| 任务跟踪 | TodoWrite | write_todos |

**粗体** = Gemini 独特优势

---

## 工具限制

### 使用 allowed-tools

在设置或命令行限制可用工具：

```bash
pgemini --allowed-tools "read_file,glob" -p "找配置文件" -o text
```

### 在设置中

```json
{
  "security": {
    "allowedTools": ["read_file", "list_directory", "glob"]
  }
}
```

---

## 最佳实践

### 何时使用特定工具

**google_web_search：**
- 需要当前/最近信息
- 检查最新版本
- 查找文档更新
- 社区解决方案

**codebase_investigator：**
- 新代码库
- 理解复杂系统
- 发现隐藏依赖
- 创建文档

**save_memory：**
- 循环项目上下文
- 用户偏好
- 自定义约定

### 工具组合模式

**研究 → 实现：**
```bash
pgemini -p "使用 Google Search 找 [主题] 最佳实践，然后实现" --yolo -o text
```

**分析 → 报告：**
```bash
pgemini -p "使用 codebase_investigator 分析项目，然后写总结报告" --yolo -o text
```

**搜索 → 读取 → 修改：**
```bash
pgemini -p "找所有使用废弃 API 的文件，读取它们，并建议更新" -o text
```