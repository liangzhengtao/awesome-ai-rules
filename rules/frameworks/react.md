# React AI 编程规则

> Last updated: 2026 | Targets: React 19+

## 核心原则

- 使用 React 19+ 特性（Server Components, `use()` hook, Server Actions）
- 优先使用函数组件和 Hooks
- 利用 React Compiler 自动优化（无需手动 memo）
- 遵循单一职责原则
- 保持组件纯净和可预测
- 优先使用 TypeScript
- Server Components 为默认，Client Components 仅在需要交互时使用

## 代码风格

### 命名规范
- 组件使用 PascalCase：`UserProfile`, `NavigationBar`
- Hooks 使用 `use` 前缀：`useAuth`, `useFetch`
- 工具函数使用 camelCase：`formatDate`, `validateEmail`
- 常量使用 UPPER_SNAKE_CASE：`API_BASE_URL`, `MAX_RETRY_COUNT`
- CSS Modules 文件使用 PascalCase：`UserProfile.module.css`
- Server Actions 文件使用 `actions.ts` 并以 `'use server'` 标记

### 文件结构
```
src/
├── components/          # 可复用组件
│   ├── Button/
│   │   ├── Button.tsx
│   │   ├── Button.test.tsx
│   │   ├── Button.stories.tsx
│   │   └── index.ts
├── hooks/               # 自定义 Hooks
├── actions/             # Server Actions（标记 'use server'）
├── pages/               # 页面组件
├── services/            # API 服务
├── stores/              # 状态管理
├── types/               # TypeScript 类型
├── utils/               # 工具函数
└── styles/              # 全局样式
```

## React 19 Features

### Server Components（稳定版）
```typescript
// ✅ Server Component（默认，无需标记）
// 可以直接访问数据库、文件系统等服务端资源
async function UserProfile({ userId }: { userId: string }) {
  const user = await db.user.findUnique({ where: { id: userId } });
  const posts = await db.post.findMany({ where: { authorId: userId } });

  return (
    <div>
      <h1>{user.name}</h1>
      <PostList posts={posts} /> {/* Client Component 处理交互 */}
    </div>
  );
}
```

### `use()` Hook
```typescript
// ✅ 在渲染时读取 Promise 和 Context，支持条件调用
function Comments({ commentsPromise }: { commentsPromise: Promise<Comment[]> }) {
  // use() 会暂停组件直到 Promise 解析完成
  const comments = use(commentsPromise);

  return (
    <ul>
      {comments.map((comment) => (
        <li key={comment.id}>{comment.text}</li>
      ))}
    </ul>
  );
}

// ✅ use() 可以在条件语句和循环中使用（区别于其他 Hooks）
function DataDisplay({ resource }: { resource: Resource }) {
  if (resource.type === 'user') {
    const user = use(resource.data); // ✅ 条件调用合法
    return <span>{user.name}</span>;
  }
  return null;
}
```

### Server Actions（`'use server'`）
```typescript
// ✅ 定义 Server Action
// actions/user.ts
'use server';

export async function createUser(formData: FormData) {
  const name = formData.get('name') as string;
  const email = formData.get('email') as string;

  await db.user.create({ data: { name, email } });
  revalidatePath('/users');
  redirect('/users');
}

// ✅ 在组件中使用 Server Action
import { createUser } from '@/actions/user';

function CreateUserForm() {
  return (
    <form action={createUser}>
      <input name="name" required />
      <input name="email" type="email" required />
      <button type="submit">Create</button>
    </form>
  );
}
```

### `useOptimistic` — 乐观更新
```typescript
// ✅ 在等待服务端响应时立即更新 UI
function TodoList({ todos }: { todos: Todo[] }) {
  const [optimisticTodos, addOptimisticTodo] = useOptimistic(
    todos,
    (state, newTodo: string) => [
      ...state,
      { id: crypto.randomUUID(), text: newTodo, completed: false },
    ]
  );

  async function handleAddTodo(formData: FormData) {
    const text = formData.get('todo') as string;
    addOptimisticTodo(text); // 立即显示新 todo
    await createTodo(text);  // 服务端完成后会用真实数据替换
  }

  return (
    <div>
      <form action={handleAddTodo}>
        <input name="todo" required />
        <button type="submit">Add</button>
      </form>
      <ul>
        {optimisticTodos.map((todo) => (
          <li key={todo.id}>{todo.text}</li>
        ))}
      </ul>
    </div>
  );
}
```

### `useActionState` — 表单状态管理
```typescript
// ✅ 管理 Server Action 的状态（pending、返回值、错误）
import { useActionState } from 'react';
import { createUser } from '@/actions/user';

function CreateUserForm() {
  const [state, formAction, isPending] = useActionState(
    createUser,
    { error: null, success: false }
  );

  return (
    <form action={formAction}>
      <input name="name" required />
      <input name="email" type="email" required />
      <button type="submit" disabled={isPending}>
        {isPending ? 'Creating...' : 'Create'}
      </button>
      {state.error && <p className="error">{state.error}</p>}
      {state.success && <p className="success">Created!</p>}
    </form>
  );
}

// ✅ 对应的 Server Action 需要接收 prevState
'use server';

export async function createUser(prevState: any, formData: FormData) {
  const name = formData.get('name') as string;
  const email = formData.get('email') as string;

  if (!name || !email) {
    return { error: 'All fields required', success: false };
  }

  await db.user.create({ data: { name, email } });
  return { error: null, success: true };
}
```

### `ref` 作为普通 prop
```typescript
// ✅ React 19 中 ref 可以直接作为 prop 传递，无需 forwardRef
interface InputProps {
  label: string;
  ref: React.Ref<HTMLInputElement>;
}

function Input({ label, ref }: InputProps) {
  return (
    <label>
      {label}
      <input ref={ref} />
    </label>
  );
}

// ✅ 使用
function Form() {
  const inputRef = useRef<HTMLInputElement>(null);
  return <Input label="Name" ref={inputRef} />;
}

// ⚠️ forwardRef 仍然兼容，但新代码不需要了
```

### Metadata 支持（`<head>` 管理）
```typescript
// ✅ React 19 原生支持在组件中渲染 <title>、<meta>、<link> 等标签
// 框架（如 Next.js）会自动将它们提升到 <head>
function BlogPost({ post }: { post: Post }) {
  return (
    <article>
      <title>{post.title} | My Blog</title>
      <meta name="description" content={post.excerpt} />
      <meta property="og:title" content={post.title} />
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </article>
  );
}
```

## 最佳实践

### 组件设计
```typescript
// ✅ 好的写法 — Server Component 为默认
interface ButtonProps {
  variant?: 'primary' | 'secondary' | 'danger';
  size?: 'sm' | 'md' | 'lg';
  isLoading?: boolean;
  children: React.ReactNode;
  onClick?: () => void;
}

export const Button: React.FC<ButtonProps> = ({
  variant = 'primary',
  size = 'md',
  isLoading = false,
  children,
  onClick,
}) => {
  return (
    <button
      className={cn(styles.button, styles[variant], styles[size])}
      disabled={isLoading}
      onClick={onClick}
    >
      {isLoading ? <Spinner /> : children}
    </button>
  );
};
```

### Hooks 使用
```typescript
// ✅ 自定义 Hook 示例
function useApi<T>(url: string) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    const controller = new AbortController();

    async function fetchData() {
      try {
        setLoading(true);
        const response = await fetch(url, {
          signal: controller.signal,
        });
        if (!response.ok) throw new Error('Failed to fetch');
        const result = await response.json();
        setData(result);
      } catch (err) {
        if (err instanceof Error && err.name !== 'AbortError') {
          setError(err);
        }
      } finally {
        setLoading(false);
      }
    }

    fetchData();
    return () => controller.abort();
  }, [url]);

  return { data, loading, error };
}

// ✅ useTransition — 低优先级更新（不阻塞输入）
function SearchFilter({ items }: { items: Item[] }) {
  const [query, setQuery] = useState('');
  const [filteredItems, setFilteredItems] = useState(items);
  const [isPending, startTransition] = useTransition();

  function handleChange(e: React.ChangeEvent<HTMLInputElement>) {
    const value = e.target.value;
    setQuery(value); // 高优先级：立即更新输入框
    startTransition(() => {
      // 低优先级：过滤结果可以延迟
      setFilteredItems(items.filter((item) =>
        item.name.toLowerCase().includes(value.toLowerCase())
      ));
    });
  }

  return (
    <div>
      <input value={query} onChange={handleChange} />
      {isPending && <Spinner />}
      <ItemList items={filteredItems} />
    </div>
  );
}

// ✅ useDeferredValue — 延迟非紧急渲染
function SearchResults({ query }: { query: string }) {
  const deferredQuery = useDeferredValue(query);
  const isStale = query !== deferredQuery;

  return (
    <div style={{ opacity: isStale ? 0.7 : 1 }}>
      <Results query={deferredQuery} />
    </div>
  );
}
```

### `<Suspense>` 流式渲染
```typescript
// ✅ 使用 Suspense 实现渐进式页面加载
function Page() {
  return (
    <main>
      <h1>Dashboard</h1>
      {/* 头部立即显示，数据部分流式加载 */}
      <Suspense fallback={<StatsSkeleton />}>
        <Stats />
      </Suspense>
      <Suspense fallback={<ChartSkeleton />}>
        <Chart />
      </Suspense>
      <Suspense fallback={<TableSkeleton />}>
        <RecentActivity />
      </Suspense>
    </main>
  );
}

// ✅ Server Components 中的异步数据获取自然配合 Suspense
async function Stats() {
  const stats = await fetchStats(); // 会暂停直到数据就绪
  return <div>{/* 渲染统计 */}</div>;
}
```

### 状态管理
- 本地状态：`useState`
- 复杂本地状态：`useReducer`
- 表单状态（Server Actions）：`useActionState`
- 乐观更新：`useOptimistic`
- 全局状态：Zustand / Jotai（轻量级）
- 服务端状态：TanStack Query / SWR

### React Compiler（自动优化）
```typescript
// ✅ React Compiler（React 19+）自动处理 memoization
// 无需手动使用 useMemo、useCallback、React.memo
// 编译器会自动分析依赖并优化重渲染

// 旧写法（手动优化，React 18 时代）
const MemoizedChild = React.memo(({ data, onClick }: Props) => {
  const sorted = useMemo(() => data.sort(), [data]);
  const handleClick = useCallback(() => onClick(sorted), [onClick, sorted]);
  return <div onClick={handleClick}>{sorted.length} items</div>;
});

// 新写法（React 19 + React Compiler，直接写即可）
function Child({ data, onClick }: Props) {
  const sorted = data.sort();
  const handleClick = () => onClick(sorted);
  return <div onClick={handleClick}>{sorted.length} items</div>;
}
// React Compiler 自动分析依赖，等效于手写 useMemo/useCallback/memo

// ⚠️ 确保代码符合 React 规范（纯函数、无副作用渲染）
// 以便编译器正确分析
```

### 性能优化
```typescript
// ✅ 如果未启用 React Compiler，仍需手动优化
const ExpensiveComponent = React.memo(({ data }: Props) => {
  return <div>{/* 复杂渲染逻辑 */}</div>;
});

// ✅ 使用 useMemo 缓存计算结果
const sortedItems = useMemo(() => {
  return items.sort((a, b) => a.name.localeCompare(b.name));
}, [items]);

// ✅ 使用 useCallback 缓存回调函数
const handleClick = useCallback(() => {
  onClick(id);
}, [onClick, id]);
```

## 常见陷阱

### ❌ 避免
```typescript
// ❌ 在循环中使用 index 作为 key
{items.map((item, index) => (
  <Item key={index} {...item} />
))}

// ❌ 在 useEffect 中缺少依赖
useEffect(() => {
  fetchData(id);
}, []); // 缺少 id 依赖

// ❌ 直接修改 state
const [user, setUser] = useState({ name: 'John' });
user.name = 'Jane'; // 错误！
setUser({ ...user, name: 'Jane' }); // 正确

// ❌ 在 Server Component 中使用 Hooks
async function Page() {
  const [count, setCount] = useState(0); // 错误！Server Component 不能用 Hooks
  return <div>{count}</div>;
}

// ❌ 混淆 'use server' 和 'use client'
'use server';
function Component() { } // 错误！'use server' 用于 Action，不是组件标记

// ❌ 将 Server Action 传递给 Client Component 的非 action prop
'use server';
export async function getUser() { return await db.user.findMany(); }

'use client';
function UserList({ fetchUsers }: { fetchUsers: () => Promise<User[]> }) {
  // 错误！Server Action 只能用于 form action 或被 use() 调用
  useEffect(() => { fetchUsers(); }, []);
}
```

### ✅ 推荐
```typescript
// ✅ 使用唯一 id 作为 key
{items.map((item) => (
  <Item key={item.id} {...item} />
))}

// ✅ 正确的 useEffect 依赖
useEffect(() => {
  fetchData(id);
}, [id]);

// ✅ 使用函数式更新
setUser((prev) => ({ ...prev, name: 'Jane' }));

// ✅ 需要交互时使用 'use client' 标记
'use client';
function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}

// ✅ Server Component 获取数据，Client Component 处理交互
async function Page() {
  const data = await fetchData(); // Server Component
  return <InteractiveComponent data={data} />; // Client Component
}
```

## 测试

```typescript
// 组件测试示例
import { render, screen, fireEvent } from '@testing-library/react';

describe('Button', () => {
  it('renders correctly', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByRole('button')).toHaveTextContent('Click me');
  });

  it('calls onClick when clicked', () => {
    const handleClick = vi.fn();
    render(<Button onClick={handleClick}>Click me</Button>);
    fireEvent.click(screen.getByRole('button'));
    expect(handleClick).toHaveBeenCalledOnce();
  });
});
```

## 依赖推荐

- **构建工具**: Vite / Next.js / Remix
- **类型检查**: TypeScript 5+
- **样式方案**: Tailwind CSS / CSS Modules
- **状态管理**: Zustand / Jotai / Redux Toolkit
- **数据获取**: TanStack Query / SWR
- **表单处理**: React Hook Form + Zod
- **测试**: Vitest + React Testing Library
- **代码规范**: ESLint + Prettier

## 项目特定规则

> 💡 在下方添加你的项目特定规则

```markdown
## 项目配置

- 项目名称：
- 包管理器：pnpm / npm / yarn
- 状态管理：
- 样式方案：
- API 方案：
- React Compiler：已启用 / 未启用
```
