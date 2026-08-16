# Next.js AI 编程规则

> Last updated: 2026 | Targets: Next.js 15+ (App Router)

## 核心原则

- 使用 App Router（Next.js 15+）
- 优先使用 Server Components（默认）
- 合理使用 Client Components（需要交互、浏览器 API 时）
- 利用 React Server Actions 处理表单和数据变更
- `params`、`cookies()`、`headers()` 均为异步 API，必须 `await`
- 遵循 Next.js 约定和文件结构
- 优先使用 Partial Prerendering (PPR) 提升首屏性能

## 代码风格

### 文件结构
```
app/
├── (auth)/                # 路由组
│   ├── login/
│   │   └── page.tsx
│   └── layout.tsx
├── (dashboard)/           # 路由组
│   ├── dashboard/
│   │   └── page.tsx
│   ├── layout.tsx
│   └── loading.tsx
├── api/                   # API Routes
│   └── users/
│       └── route.ts
├── layout.tsx             # 根布局
├── page.tsx               # 首页
├── loading.tsx            # 全局 loading
├── error.tsx              # 全局错误
├── not-found.tsx          # 404 页面
└── template.tsx           # 模板（每次导航重新挂载）
```

### 命名规范
- 页面文件：`page.tsx`
- 布局文件：`layout.tsx`
- 加载状态：`loading.tsx`
- 错误处理：`error.tsx`
- API 路由：`route.ts`
- Server Actions：`actions.ts`

## 最佳实践

### Server Components vs Client Components

```typescript
// ✅ Server Components（默认，无需标记）
// app/page.tsx
export default async function HomePage() {
  const posts = await fetch('https://api.example.com/posts', {
    cache: 'force-cache', // 静态数据
  }).then(res => res.json());

  return (
    <main>
      <h1>Latest Posts</h1>
      <PostList posts={posts} />
    </main>
  );
}

// ✅ Client Components（需要交互时）
// components/PostList.tsx
'use client';

import { useState } from 'react';

export function PostList({ posts }: { posts: Post[] }) {
  const [filter, setFilter] = useState('all');

  return (
    <div>
      <select value={filter} onChange={(e) => setFilter(e.target.value)}>
        <option value="all">All</option>
        <option value="published">Published</option>
      </select>
      {/* ... */}
    </div>
  );
}
```

### 异步 params（Next.js 15+）

```typescript
// ✅ Next.js 15+ 中 params 是 Promise，必须 await
// app/posts/[id]/page.tsx
export default async function PostPage({
  params,
}: {
  params: Promise<{ id: string }>;
}) {
  const { id } = await params; // ✅ 必须 await
  const post = await getPost(id);

  return <article>{post.title}</article>;
}

// ✅ generateMetadata 同样需要 await params
export async function generateMetadata({
  params,
}: {
  params: Promise<{ id: string }>;
}): Promise<Metadata> {
  const { id } = await params;
  const post = await getPost(id);

  return {
    title: post.title,
    description: post.excerpt,
  };
}

// ✅ generateStaticParams 保持不变
export async function generateStaticParams() {
  const posts = await getAllPosts();
  return posts.map((post) => ({ id: post.id }));
}

// ❌ 错误写法（Next.js 14 旧语法）
export default function PostPage({ params }: { params: { id: string } }) {
  const post = await getPost(params.id); // 不要这样写！
}
```

### 异步 cookies() 和 headers()

```typescript
// ✅ Next.js 15+ 中 cookies() 和 headers() 是异步的
import { cookies, headers } from 'next/headers';

export default async function Page() {
  const cookieStore = await cookies(); // ✅ 必须 await
  const token = cookieStore.get('token')?.value;

  const headersList = await headers(); // ✅ 必须 await
  const userAgent = headersList.get('user-agent');

  return <div>User: {userAgent}</div>;
}

// ❌ 错误写法（旧语法）
export default async function Page() {
  const token = cookies().get('token')?.value; // 不要这样写！
}
```

### 数据获取

```typescript
// ✅ Server Component 中直接 fetch
async function getData() {
  const res = await fetch('https://api.example.com/data', {
    next: { revalidate: 3600 }, // ISR: 每小时重新验证
  });

  if (!res.ok) {
    throw new Error('Failed to fetch data');
  }

  return res.json();
}

// ✅ 使用 use cache（Next.js 15 canary，替代 unstable_cache）
// 标记整个函数或组件为可缓存
import { use } from 'react';

// 在 Server Component 中使用 'use cache' 指令
async function getCachedPosts() {
  'use cache'; // 标记为可缓存函数
  // 可以设置缓存标签和过期时间
  cacheTag('posts');
  cacheLife('minutes'); // 预设：seconds / minutes / hours / days

  return await db.post.findMany();
}

// ✅ 旧方案 unstable_cache 仍然可用（但推荐 use cache）
import { unstable_cache } from 'next/cache';

const getCachedData = unstable_cache(
  async (id: string) => {
    return await db.post.findUnique({ where: { id } });
  },
  ['post-detail'],
  { revalidate: 60, tags: ['posts'] }
);
```

### Partial Prerendering (PPR)

```typescript
// ✅ Partial Prerendering：静态外壳 + 动态内容流式插入
// 在 next.config.ts 中启用：
// experimental: { ppr: 'incremental' }

// app/dashboard/page.tsx
// 这个页面的静态部分会在构建时生成，动态部分在请求时流式渲染
export default async function DashboardPage() {
  // 静态部分：导航、布局等在构建时渲染
  return (
    <main>
      <h1>Dashboard</h1>
      <nav>
        <Sidebar /> {/* 静态 */}
      </nav>
      <Suspense fallback={<StatsSkeleton />}>
        <Stats /> {/* 动态：每次请求时渲染 */}
      </Suspense>
      <Suspense fallback={<ChartSkeleton />}>
        <Chart /> {/* 动态：每次请求时渲染 */}
      </Suspense>
    </main>
  );
}

async function Stats() {
  const data = await fetchStats(); // 每次请求时获取
  return <div>{/* 渲染统计 */}</div>;
}

// ✅ 路由级别配置 PPR
// app/dashboard/page.tsx
export const experimental_ppr = true; // 启用 PPR

// app/api/route.ts（不适用 PPR，纯动态）
export const experimental_ppr = false; // 禁用 PPR
```

### Server Actions + useActionState

```typescript
// ✅ Server Actions 配合 useActionState 管理表单状态
// app/posts/actions.ts
'use server';

import { revalidatePath } from 'next/cache';

export async function createPost(
  prevState: { error: string | null; success: boolean },
  formData: FormData
) {
  const title = formData.get('title') as string;
  const content = formData.get('content') as string;

  // 验证
  if (!title || title.length < 3) {
    return { error: 'Title must be at least 3 characters', success: false };
  }

  // 创建
  await db.post.create({
    data: { title, content },
  });

  // 重新验证缓存
  revalidatePath('/posts');

  return { error: null, success: true };
}

// ✅ 在组件中使用
// app/posts/new/page.tsx
'use client';

import { useActionState } from 'react';
import { createPost } from '../actions';

export default function NewPostPage() {
  const [state, formAction, isPending] = useActionState(createPost, {
    error: null,
    success: false,
  });

  return (
    <form action={formAction}>
      <input name="title" required />
      <textarea name="content" required />
      <button type="submit" disabled={isPending}>
        {isPending ? 'Creating...' : 'Create Post'}
      </button>
      {state.error && <p className="text-red-500">{state.error}</p>}
      {state.success && <p className="text-green-500">Post created!</p>}
    </form>
  );
}
```

### Middleware

```typescript
// ✅ middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  // 认证检查
  const token = request.cookies.get('token')?.value;

  if (!token && request.nextUrl.pathname.startsWith('/dashboard')) {
    return NextResponse.redirect(new URL('/login', request.url));
  }

  // 添加自定义 header
  const response = NextResponse.next();
  response.headers.set('x-custom-header', 'value');

  return response;
}

export const config = {
  matcher: ['/dashboard/:path*', '/api/:path*'],
};
```

### SEO 优化

```typescript
// ✅ Metadata API
import type { Metadata } from 'next';

export const metadata: Metadata = {
  title: {
    template: '%s | My Site',
    default: 'My Site',
  },
  description: 'My awesome site',
  openGraph: {
    title: 'My Site',
    description: 'My awesome site',
    images: ['/og-image.png'],
  },
};

// ✅ 动态 Metadata — params 必须 await
export async function generateMetadata({
  params,
}: {
  params: Promise<{ id: string }>;
}): Promise<Metadata> {
  const { id } = await params;
  const post = await getPost(id);

  return {
    title: post.title,
    description: post.excerpt,
  };
}
```

## 性能优化

### 图片优化
```typescript
import Image from 'next/image';

// ✅ 使用 Next.js Image 组件
<Image
  src="/hero.jpg"
  alt="Hero"
  width={800}
  height={600}
  priority        // 首屏图片
  placeholder="blur"
  blurDataURL="data:image/jpeg;base64,..."
/>
```

### 字体优化
```typescript
import { Inter, Roboto } from 'next/font/google';

const inter = Inter({
  subsets: ['latin'],
  display: 'swap',
  variable: '--font-inter',
});

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en" className={inter.variable}>
      <body>{children}</body>
    </html>
  );
}
```

## 常见陷阱

### ❌ 避免
```typescript
// ❌ 不 await params（Next.js 15+ 强制异步）
export default function Page({ params }: { params: { id: string } }) {
  const post = await getPost(params.id); // 错误！
}

// ❌ 不 await cookies() / headers()
import { cookies } from 'next/headers';
export default async function Page() {
  const token = cookies().get('token'); // 错误！
}

// ❌ 在 Server Component 中使用 useState
'use client'; // 缺少这行就会报错
export default function Page() {
  const [count, setCount] = useState(0);
  return <div>{count}</div>;
}

// ❌ 在 Client Component 中直接访问数据库
'use client';
export default function Page() {
  const data = await db.query(); // 错误！不能在客户端访问数据库
}

// ❌ 在 Server Component 中使用事件处理
export default function Page() {
  return (
    <button onClick={() => console.log('clicked')}>Click</button> // 错误！
  );
}

// ❌ 混淆 'use server' 和 'use client'
'use server';
export default function Component() {} // 错误！'use server' 用于 Actions，不是组件
```

### ✅ 推荐
```typescript
// ✅ 正确使用 async params
export default async function Page({
  params,
}: {
  params: Promise<{ id: string }>;
}) {
  const { id } = await params;
  const post = await getPost(id);
  return <article>{post.title}</article>;
}

// ✅ 正确使用 async cookies
import { cookies } from 'next/headers';
export default async function Page() {
  const cookieStore = await cookies();
  const token = cookieStore.get('token')?.value;
  return <div>Token: {token}</div>;
}

// ✅ 需要交互时使用 Client Component
'use client';
export default function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}

// ✅ Server Component 获取数据，Client Component 处理交互
export default async function Page() {
  const data = await fetchData(); // Server
  return <InteractiveComponent data={data} />; // Client
}
```

## 依赖推荐

- **UI 组件**: shadcn/ui / Radix UI
- **样式**: Tailwind CSS
- **状态管理**: Zustand / Jotai
- **表单**: React Hook Form + Zod（或直接用 Server Actions + useActionState）
- **认证**: NextAuth.js v5 / Clerk
- **数据库**: Prisma / Drizzle ORM
- **部署**: Vercel

## 项目特定规则

> 💡 在下方添加你的项目特定规则

```markdown
## 项目配置

- 项目名称：
- 数据库：
- 认证方案：
- 部署平台：
- 包管理器：
- PPR：已启用 / 未启用
- React Compiler：已启用 / 未启用
```
