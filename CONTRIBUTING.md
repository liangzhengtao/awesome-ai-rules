# 贡献指南

感谢你对 Awesome AI Rules 项目的关注！我们欢迎任何形式的贡献。

## 如何贡献

### 1. 提交新的规则模板

#### Fork 项目

```bash
# Fork 项目到你的 GitHub 账号
# 然后克隆到本地
git clone https://github.com/OWNER/awesome-ai-rules.git
cd awesome-ai-rules
```

#### 创建新分支

```bash
git checkout -b add-react-native-rules
```

#### 添加规则文件

```bash
# 在对应的目录下创建规则文件
# 框架规则：rules/frameworks/
# 场景规则：rules/scenarios/
# 工具规则：rules/tools/

# 例如添加 React Native 框架规则
touch rules/frameworks/react-native.md
```

#### 编写规则

请参考 [规则编写指南](#规则编写指南) 编写高质量的规则。

#### 提交 PR

```bash
git add .
git commit -m "feat: add React Native framework rules"
git push origin add-react-native-rules
```

然后在 GitHub 上创建 Pull Request。

### 2. 改进现有规则

1. 找到需要改进的规则文件
2. 提出改进建议或直接修改
3. 提交 PR 并说明改进原因

### 3. 报告问题

如果发现问题，请在 Issues 中报告：

1. 使用清晰的标题
2. 描述问题的具体表现
3. 提供复现步骤（如果适用）
4. 提供你的环境信息

### 4. 添加新的工具支持

如果你想为新的 AI 工具添加规则：

1. 在 `rules/tools/` 目录下创建新文件
2. 按照现有工具规则的格式编写
3. 更新 README.md 中的工具列表

## 规则编写指南

### 文件格式

每份规则文件应遵循以下格式：

```markdown
# [技术栈/场景/工具] AI 编程规则

## 核心原则

- 简洁列出最重要的原则

## 代码风格

### 命名规范
- 规范 1
- 规范 2

### 文件结构
```
示例结构
```

## 最佳实践

### [主题 1]
```代码示例```

### [主题 2]
```代码示例```

## 常见陷阱

### ❌ 避免
```错误示例```

### ✅ 推荐
```正确示例```

## 依赖推荐

- **类别**: 推荐库 1 / 推荐库 2

## 项目特定规则

> 💡 在下方添加你的项目特定规则

```markdown
## 项目配置

- 项目名称：
- 技术栈：
```
```

### 编写原则

1. **实用性优先**
   - 提供实际可用的代码示例
   - 包含真实的最佳实践
   - 避免理论性的空洞内容

2. **清晰简洁**
   - 使用简洁的语言
   - 代码示例要清晰易懂
   - 避免冗长的解释

3. **一致性**
   - 遵循现有的格式和风格
   - 使用统一的术语
   - 保持代码风格一致

4. **完整性**
   - 覆盖常见的使用场景
   - 包含错误处理
   - 提供测试示例

### 代码示例要求

1. **可运行**
   - 代码示例应该是可运行的
   - 包含必要的导入和依赖
   - 避免伪代码

2. **最佳实践**
   - 展示推荐的写法
   - 同时展示需要避免的写法
   - 解释为什么推荐某种写法

3. **类型安全**
   - TypeScript 代码必须有类型定义
   - Python 代码建议使用 type hints
   - 展示类型安全的写法

### 质量检查清单

在提交 PR 之前，请检查：

- [ ] 规则文件遵循统一格式
- [ ] 代码示例可运行
- [ ] 包含错误处理示例
- [ ] 包含测试示例
- [ ] 命名规范清晰
- [ ] 文件结构合理
- [ ] 没有拼写和语法错误
- [ ] 代码风格一致

## 提交规范

### Commit Message 格式

使用 [Conventional Commits](https://www.conventionalcommits.org/) 规范：

```
<type>(<scope>): <subject>

<body>

<footer>
```

#### Type 类型

- `feat`: 新功能
- `fix`: 修复问题
- `docs`: 文档更新
- `style`: 代码风格调整（不影响功能）
- `refactor`: 代码重构
- `test`: 测试相关
- `chore`: 构建/工具相关

#### 示例

```
feat(react): add React Native framework rules

- Add React Native specific rules
- Include best practices for navigation
- Add testing guidelines

Closes #123
```

### PR 规范

1. **标题清晰**
   - 使用清晰的标题描述改动
   - 遵循 Commit Message 格式

2. **描述详细**
   - 说明改动的原因
   - 列出主要改动
   - 提供相关 Issue 链接

3. **小步提交**
   - 每个 PR 只做一件事
   - 避免大型 PR
   - 保持 PR 简洁

## 代码审查

### 审查要点

1. **内容质量**
   - 规则是否实用
   - 代码示例是否正确
   - 是否有遗漏的场景

2. **格式规范**
   - 是否遵循统一格式
   - Markdown 语法是否正确
   - 代码块是否正确标记

3. **一致性**
   - 与现有规则风格是否一致
   - 术语使用是否一致
   - 命名规范是否一致

### 审查流程

1. 自动检查（如果配置了 CI）
2. 维护者审查
3. 社区反馈
4. 合并

## 社区准则

### 行为准则

- 尊重他人
- 建设性反馈
- 包容不同观点
- 避免人身攻击

### 沟通规范

- 使用清晰的语言
- 提供具体的例子
- 保持专业态度
- 及时回复

## 常见问题

### Q: 我可以提交商业项目的规则吗？

A: 可以，但请确保规则是通用的，不包含商业机密。

### Q: 我可以翻译现有的规则吗？

A: 欢迎！请在 `rules/` 目录下创建语言子目录，如 `rules/zh/`。

### Q: 如何报告安全问题？

A: 请通过私信联系维护者，不要在公开 Issue 中报告安全问题。

### Q: 我可以添加自己的工具规则吗？

A: 当然！只要你的工具是 AI 编程助手，我们都欢迎。

## 联系方式

- GitHub Issues: [项目 Issues 页面]
- Email: [维护者邮箱]

## 致谢

感谢所有贡献者的努力！

[![Contributors](https://contrib.rocks/image?repo=OWNER/awesome-ai-rules)](https://github.com/OWNER/awesome-ai-rules/graphs/contributors)
