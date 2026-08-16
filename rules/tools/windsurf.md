# Windsurf AI 编程规则
> Last updated: 2026 | Targets: Windsurf (Codeium)

## 核心原则

- 使用 `.windsurfrules` 文件配置项目级规则
- 善用 Windsurf 的代码补全和内联建议
- 利用 Chat 面板进行复杂任务
- 保持规则简洁明确

## 配置文件

### .windsurfrules 示例

```markdown
# Windsurf 项目规则

## 技术栈
- 框架：[框架名称]
- 语言：[编程语言]
- 数据库：[数据库]

## 代码规范
- 使用 TypeScript strict mode
- 使用 ESLint + Prettier
- 优先使用 const
- 使用 early return 减少嵌套

## 命名规范
- 组件：PascalCase
- 函数/变量：camelCase
- 常量：UPPER_SNAKE_CASE
- 文件：kebab-case

## 特殊指令
- 不要使用 any 类型
- 所有异步操作必须处理错误
- 组件必须有 Props 类型定义
```

## 最佳实践

### 规则配置

```markdown
# 保持规则简洁
- 每条规则不超过一行
- 使用明确的指令
- 避免模糊的描述

# 分层组织
## 核心规则（必须遵守）
## 推荐规则（建议遵守）
## 可选规则（根据情况）
```

### 常用提示

```markdown
# 代码生成
请实现一个 [功能描述]

# 代码审查
请审查这段代码的 [关注点]

# 重构
请重构这段代码，目标是 [目标]
```

## 常见陷阱

### ❌ 避免
```markdown
# ❌ 规则太模糊
写好的代码

# ❌ 规则太长
（超过 500 行的规则文件）
```

### ✅ 推荐
```markdown
# ✅ 规则具体明确
使用 const 声明不会重新赋值的变量

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
