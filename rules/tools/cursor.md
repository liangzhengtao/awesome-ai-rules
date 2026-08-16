# Cursor AI 编程规则
> Last updated: 2026 | Targets: Cursor 0.40+

## 核心原则

- 使用 `.cursorrules` 文件配置项目级规则
- 善用 Cmd+K 快速编辑
- 利用 Composer 进行多文件编辑
- 使用 @ 符号引用上下文
- 保持规则简洁明确

## 配置文件

### .cursorrules 示例

```markdown
# 项目规则

## 技术栈
- 框架：Next.js 14 (App Router)
- 语言：TypeScript (strict mode)
- 样式：Tailwind CSS
- 数据库：PostgreSQL + Prisma
- 认证：NextAuth.js

## 代码规范
- 使用函数组件和 Hooks
- 优先使用 Server Components
- 使用 TypeScript 严格模式
- 遵循 ESLint + Prettier 配置

## 文件结构
- 页面组件：app/[route]/page.tsx
- 布局组件：app/[route]/layout.tsx
- API 路由：app/api/[route]/route.ts
- 通用组件：components/[name].tsx
- Hooks：hooks/use-[name].ts

## 命名规范
- 组件：PascalCase
- 函数/变量：camelCase
- 常量：UPPER_SNAKE_CASE
- 文件：kebab-case

## 特殊指令
- 不要使用 `any` 类型
- 不要使用 `// @ts-ignore`
- 所有异步操作必须处理错误
- 组件必须有 Props 类型定义
- API 路由必须有请求验证
```

## 最佳实践

### 使用 @ 符号引用上下文

```
# 在聊天中使用 @ 引用文件
@src/components/UserCard.tsx 请添加 loading 状态

# 引用整个文件夹
@src/components 请检查是否有未使用的组件

# 引用文档
@docs/api.md 请根据文档实现 API

# 引用网页
@https://ui.shadcn.com/docs/components/button 请按照文档实现 Button 组件
```

### Composer 多文件编辑

```
# 使用 Composer 进行跨文件修改
请实现用户注册功能：
1. 创建注册表单组件
2. 添加 API 路由
3. 实现邮箱验证
4. 更新数据库 schema
```

### 快速编辑技巧

```
# 选中代码后按 Cmd+K
请重构这个函数，提取出验证逻辑

# 在终端中按 Cmd+K
请解释这个错误信息并提供解决方案

# 在编辑器中按 Cmd+L 打开聊天
请解释这段代码的作用
```

## 项目级配置

### .cursor/settings.json

```json
{
  "cursor.cppEnabled": true,
  "cursor.chat.enableCodeGeneration": true,
  "cursor.chat.enableLongContext": true,
  "cursor.codeGeneration.enableComments": true,
  "cursor.codeGeneration.enableTypeScript": true
}
```

### .cursorignore

```
# 忽略不需要索引的文件
node_modules/
.next/
dist/
coverage/
*.min.js
*.min.css
```

## 常用提示模板

### 代码生成

```markdown
请生成一个 [功能描述] 组件/函数：

要求：
- 使用 [技术栈]
- 遵循 [规范]
- 包含 [特性]

示例输入：[输入]
示例输出：[输出]
```

### 代码审查

```markdown
请审查这段代码：

关注点：
- 性能问题
- 安全漏洞
- 最佳实践
- 可读性改进
```

### Bug 修复

```markdown
遇到以下错误：
[错误信息]

相关代码：
[代码片段]

请分析原因并提供修复方案
```

### 重构

```markdown
请重构这段代码：

目标：
- 提高可读性
- 减少重复
- 改善性能
- 添加类型安全

保持向后兼容
```

## 工作流优化

### 1. 新功能开发

```
1. 使用 Composer 描述需求
2. 审查生成的代码
3. 使用 Cmd+K 调整细节
4. 运行测试验证
5. 使用 Cmd+L 获取改进建议
```

### 2. Bug 修复

```
1. 复制错误信息
2. 使用 @ 引用相关文件
3. 让 Cursor 分析原因
4. 应用修复方案
5. 验证修复
```

### 3. 代码审查

```
1. 选中代码块
2. 使用 Cmd+K 请求审查
3. 关注特定方面（性能、安全等）
4. 应用建议
```

## 高级技巧

### 使用 .cursorrules 分层

```markdown
# 全局规则（项目根目录）
## 通用规范
- TypeScript strict
- ESLint + Prettier

# 模块规则（子目录）
## API 模块
- 使用 Fastify
- 验证所有输入

## UI 模块
- 使用 React
- Tailwind CSS
```

### 自定义指令

```markdown
# .cursorrules 中的自定义指令

## 代码生成指令
- 生成代码时添加 JSDoc 注释
- 为函数添加参数验证
- 使用 early return 减少嵌套

## 错误处理指令
- 始终使用 try-catch 包裹异步操作
- 记录错误日志
- 返回用户友好的错误信息

## 测试指令
- 为新功能编写单元测试
- 使用 AAA 模式（Arrange-Act-Assert）
- 测试边界情况
```

## 常见陷阱

### ❌ 避免
```markdown
# ❌ 规则太模糊
写好的代码

# ❌ 规则太长
（超过 500 行的规则文件）

# ❌ 规则冲突
使用 var 声明变量
使用 const 声明变量
```

### ✅ 推荐
```markdown
# ✅ 规则具体明确
使用 const 声明不会重新赋值的变量
使用 let 声明需要重新赋值的变量
不要使用 var

# ✅ 规则分层组织
## 核心规则（必须遵守）
## 推荐规则（建议遵守）
## 可选规则（根据情况）

# ✅ 规则简洁
每条规则一行，不超过 80 字符
```

## 项目特定规则

> 💡 在下方添加你的项目特定规则

```markdown
## 项目配置

- 项目名称：
- 技术栈：
- 特殊指令：
```
