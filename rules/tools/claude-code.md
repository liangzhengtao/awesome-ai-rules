# Claude Code AI 编程规则
> Last updated: 2026 | Targets: Claude Code (Anthropic)

## 核心原则

- 使用 `CLAUDE.md` 文件配置项目规则
- 善用 @ 引用文件和上下文
- 利用 Plan Mode 进行复杂任务规划
- 使用 Todo List 追踪任务进度
- 保持对话上下文清晰

## 配置文件

### CLAUDE.md 示例

```markdown
# Claude Code 项目规则

## 项目概述
这是一个 [项目类型] 项目，使用 [技术栈]。

## 技术栈
- 框架：[框架名称]
- 语言：[编程语言]
- 数据库：[数据库]
- 部署：[部署方式]

## 代码规范

### 命名规范
- 组件：PascalCase
- 函数/变量：camelCase
- 常量：UPPER_SNAKE_CASE
- 文件：kebab-case

### 文件结构
```
src/
├── components/     # 通用组件
├── pages/          # 页面组件
├── hooks/          # 自定义 Hooks
├── services/       # API 服务
├── utils/          # 工具函数
└── types/          # TypeScript 类型
```

### 代码风格
- 使用 TypeScript strict mode
- 使用 ESLint + Prettier
- 优先使用 const
- 使用 early return 减少嵌套
- 函数不超过 50 行
- 文件不超过 300 行

## 特殊指令

### 代码生成
- 生成代码时添加 JSDoc 注释
- 为函数添加参数验证
- 使用 TypeScript 类型定义
- 处理所有可能的错误

### 测试
- 为新功能编写单元测试
- 使用 AAA 模式（Arrange-Act-Assert）
- 测试边界情况和错误情况
- 测试覆盖率不低于 80%

### 安全
- 验证所有用户输入
- 使用参数化查询防止 SQL 注入
- 对敏感数据进行加密
- 实施适当的认证和授权

### 性能
- 避免 N+1 查询
- 使用缓存减少重复计算
- 懒加载非关键资源
- 优化图片和静态资源

## 工作流

### 开发流程
1. 使用 Plan Mode 规划任务
2. 创建 Todo List 追踪进度
3. 逐个完成任务
4. 运行测试验证
5. 代码审查

### 命名约定
- 分支：feature/xxx, fix/xxx, refactor/xxx
- 提交：feat: xxx, fix: xxx, refactor: xxx
- PR：[类型] 简短描述
```

## 最佳实践

### 使用 @ 引用上下文

```markdown
# 引用文件
@src/components/Button.tsx 请添加 loading 状态

# 引用目录
@src/components 请检查是否有未使用的导出

# 引用文档
@README.md 请根据文档更新 API

# 引用错误日志
@error.log 请分析错误原因
```

### Plan Mode 使用

```markdown
# 进入 Plan Mode
请规划实现用户认证功能

# Claude 会：
1. 分析需求
2. 设计架构
3. 列出实现步骤
4. 估算工作量
5. 询问你的确认

# 退出 Plan Mode
请开始实现
```

### Todo List 管理

```markdown
# 创建任务列表
请创建一个任务列表来实现这个功能

# 更新任务状态
请将第一个任务标记为完成

# 查看进度
请显示当前任务进度
```

## 常用提示模板

### 功能开发

```markdown
请实现 [功能名称]：

需求：
- [需求 1]
- [需求 2]
- [需求 3]

技术要求：
- 使用 [技术栈]
- 遵循 [规范]
- 包含 [特性]

请先规划，然后实现。
```

### 代码审查

```markdown
请审查这段代码：

@src/path/to/file.ts

关注点：
- 代码质量
- 性能问题
- 安全漏洞
- 最佳实践

请提供具体的改进建议。
```

### Bug 修复

```markdown
遇到以下问题：

错误信息：
[错误信息]

复现步骤：
1. [步骤 1]
2. [步骤 2]
3. [步骤 3]

相关代码：
@src/path/to/file.ts

请分析原因并提供修复方案。
```

### 重构

```markdown
请重构这段代码：

@src/path/to/file.ts

目标：
- 提高可读性
- 减少重复代码
- 改善性能
- 添加类型安全

约束：
- 保持向后兼容
- 不改变外部 API
- 所有测试必须通过
```

### 测试编写

```markdown
请为这个函数编写测试：

@src/path/to/file.ts

要求：
- 测试正常流程
- 测试边界情况
- 测试错误处理
- 使用 AAA 模式
- 覆盖率 > 80%
```

## 工作流优化

### 1. 新功能开发

```
1. 描述需求，让 Claude 规划
2. 确认计划，开始实现
3. 使用 Todo List 追踪进度
4. 逐步完成每个任务
5. 运行测试验证
6. 代码审查和优化
```

### 2. Bug 修复

```
1. 描述问题和错误信息
2. @ 引用相关代码
3. 让 Claude 分析原因
4. 确认修复方案
5. 应用修复
6. 验证修复
```

### 3. 代码重构

```
1. @ 引用要重构的代码
2. 描述重构目标
3. 让 Claude 规划重构步骤
4. 逐步执行重构
5. 运行测试确保不破坏功能
6. 审查重构结果
```

## 高级技巧

### 使用多轮对话

```markdown
# 第一轮：规划
请规划实现用户认证功能

# 第二轮：确认
计划看起来不错，请开始实现

# 第三轮：实现
请继续实现下一个任务

# 第四轮：测试
请为实现的代码编写测试

# 第五轮：优化
请优化性能和代码质量
```

### 使用上下文

```markdown
# 保持上下文
之前的对话中，我们讨论了用户认证的设计。
现在请实现登录功能。

# 引用之前的代码
请基于之前实现的用户模型，添加密码重置功能。
```

### 自定义指令

```markdown
# CLAUDE.md 中的自定义指令

## 代码生成指令
- 使用 functional programming 风格
- 优先使用 map/filter/reduce
- 避免副作用
- 使用 immutable 数据

## 错误处理指令
- 使用 Result 类型处理错误
- 不要抛出异常
- 提供详细的错误信息
- 记录错误日志

## 测试指令
- 使用 Jest 测试框架
- Mock 外部依赖
- 测试异步代码
- 使用 snapshot 测试
```

## 项目特定规则

### Next.js 项目

```markdown
## Next.js 特定规则

### 路由
- 使用 App Router
- 页面组件使用 page.tsx
- 布局组件使用 layout.tsx
- 加载状态使用 loading.tsx

### 数据获取
- 优先使用 Server Components
- 使用 fetch 进行数据获取
- 使用 React Query 管理客户端状态

### 样式
- 使用 Tailwind CSS
- 使用 CSS Modules 作为备选
- 避免内联样式
```

### Python 项目

```markdown
## Python 特定规则

### 代码风格
- 遵循 PEP 8
- 使用 type hints
- 使用 Black 格式化
- 使用 isort 排序导入

### 项目结构
- 使用 src layout
- 分离 tests 目录
- 使用 pyproject.toml

### 依赖管理
- 使用 Poetry
- 锁定依赖版本
- 分离 dev 和 prod 依赖
```

## 常见陷阱

### ❌ 避免
```markdown
# ❌ 上下文过长
（一次性引用太多文件）

# ❌ 指令模糊
请优化代码

# ❌ 任务过大
请实现整个电商系统
```

### ✅ 推荐
```markdown
# ✅ 适量上下文
@src/components/Button.tsx 请优化这个组件

# ✅ 指令具体
请优化这个函数的性能，减少不必要的重渲染

# ✅ 任务分解
请实现用户登录功能的第一步：创建登录表单
```

## 项目特定规则

> 💡 在下方添加你的项目特定规则

```markdown
## 项目配置

- 项目名称：
- 技术栈：
- 特殊指令：
```
