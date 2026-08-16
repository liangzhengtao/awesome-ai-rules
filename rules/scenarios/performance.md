# Performance AI 编程规则
> Last updated: 2026 | Targets: Core Web Vitals / Vite 5+ / Next.js 15+

## 核心原则

- 先测量，后优化
- 关注关键性能指标（Core Web Vitals）
- 使用性能预算
- 持续监控和警报
- 权衡性能与代码可维护性

## 代码风格

### 性能指标
- **LCP** (Largest Contentful Paint): < 2.5s
- **FID** (First Input Delay): < 100ms
- **CLS** (Cumulative Layout Shift): < 0.1
- **TTFB** (Time to First Byte): < 800ms
- **FCP** (First Contentful Paint): < 1.8s

## 最佳实践

### 前端性能

```typescript
// ✅ 代码分割和懒加载
import { lazy, Suspense } from 'react';

const HeavyComponent = lazy(() => import('./HeavyComponent'));

function App() {
  return (
    <Suspense fallback={<Loading />}>
      <HeavyComponent />
    </Suspense>
  );
}

// ✅ 路由级别代码分割
import { createBrowserRouter } from 'react-router-dom';

const router = createBrowserRouter([
  {
    path: '/',
    lazy: () => import('./pages/Home'),
  },
  {
    path: '/dashboard',
    lazy: () => import('./pages/Dashboard'),
  },
]);

// ✅ 图片优化
import Image from 'next/image';

<Image
  src="/hero.jpg"
  alt="Hero"
  width={800}
  height={600}
  priority           // 首屏图片
  placeholder="blur"
  blurDataURL={blurHash}
  sizes="(max-width: 768px) 100vw, 50vw"
/>

// ✅ 字体优化
import { Inter } from 'next/font/google';

const inter = Inter({
  subsets: ['latin'],
  display: 'swap',        // 避免 FOIT
  preload: true,
  fallback: ['system-ui', 'sans-serif'],
});

// ✅ 预加载关键资源
<link rel="preload" href="/api/critical-data" as="fetch" crossOrigin="anonymous" />
<link rel="preconnect" href="https://fonts.googleapis.com" />
<link rel="dns-prefetch" href="https://cdn.example.com" />
```

### React 性能优化

```typescript
// ✅ 使用 React.memo 避免不必要的重渲染
const ExpensiveComponent = React.memo(({ data, onClick }: Props) => {
  return (
    <div onClick={onClick}>
      {data.map(item => (
        <Item key={item.id} {...item} />
      ))}
    </div>
  );
});

// ✅ 使用 useMemo 缓存计算结果
function DataGrid({ data, sortField, filterText }: Props) {
  const processedData = useMemo(() => {
    return data
      .filter(item => item.name.includes(filterText))
      .sort((a, b) => a[sortField] - b[sortField]);
  }, [data, sortField, filterText]);

  return <Grid data={processedData} />;
}

// ✅ 使用 useCallback 缓存回调函数
function ParentComponent() {
  const [count, setCount] = useState(0);

  const handleClick = useCallback(() => {
    setCount(prev => prev + 1);
  }, []);

  return <ChildComponent onClick={handleClick} />;
}

// ✅ 使用虚拟列表处理大数据
import { useVirtualizer } from '@tanstack/react-virtual';

function VirtualList({ items }: { items: Item[] }) {
  const parentRef = useRef<HTMLDivElement>(null);

  const virtualizer = useVirtualizer({
    count: items.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 50,
  });

  return (
    <div ref={parentRef} style={{ height: '400px', overflow: 'auto' }}>
      <div style={{ height: `${virtualizer.getTotalSize()}px` }}>
        {virtualizer.getVirtualItems().map(virtualRow => (
          <div
            key={virtualRow.key}
            style={{
              position: 'absolute',
              top: `${virtualRow.start}px`,
              height: `${virtualRow.size}px`,
              width: '100%',
            }}
          >
            {items[virtualRow.index].name}
          </div>
        ))}
      </div>
    </div>
  );
}
```

### 状态管理优化

```typescript
// ✅ Zustand - 选择性订阅
import { create } from 'zustand';

interface Store {
  users: User[];
  selectedUser: User | null;
  setUsers: (users: User[]) => void;
  selectUser: (user: User) => void;
}

const useStore = create<Store>((set) => ({
  users: [],
  selectedUser: null,
  setUsers: (users) => set({ users }),
  selectUser: (user) => set({ selectedUser: user }),
}));

// ❌ 订阅整个 store（会导致不必要的重渲染）
function UserList() {
  const store = useStore(); // 任何变化都会重渲染
  return <div>{store.users.map(...)}</div>;
}

// ✅ 只订阅需要的部分
function UserList() {
  const users = useStore(state => state.users);
  return <div>{users.map(...)}</div>;
}

// ✅ TanStack Query - 缓存和去重
import { useQuery } from '@tanstack/react-query';

function UserProfile({ userId }: { userId: string }) {
  const { data, isLoading } = useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
    staleTime: 5 * 60 * 1000,     // 5 分钟内认为数据是新鲜的
    cacheTime: 30 * 60 * 1000,    // 缓存 30 分钟
    refetchOnWindowFocus: false,   // 窗口聚焦时不重新获取
  });

  if (isLoading) return <Skeleton />;
  return <div>{data.name}</div>;
}
```

### CSS 性能

```css
/* ✅ 使用 containment */
.card {
  contain: layout style paint;
}

/* ✅ 使用 will-change 提示浏览器 */
.animated-element {
  will-change: transform;
}

/* ✅ 避免昂贵的选择器 */
/* ❌ */
div > ul > li > a > span { }

/* ✅ */
.nav-link-text { }

/* ✅ 使用 content-visibility */
.below-fold {
  content-visibility: auto;
  contain-intrinsic-size: 0 500px;
}

/* ✅ 减少重绘重排 */
/* ❌ */
function moveElement() {
  element.style.left = '10px';  // 触发重排
  element.style.top = '10px';   // 再次触发重排
}

/* ✅ */
function moveElement() {
  element.style.transform = 'translate(10px, 10px)';  // 只触发合成
}
```

### 后端性能

```typescript
// ✅ 数据库查询优化
// ❌ N+1 查询
async function getUsersWithPosts() {
  const users = await db.query('SELECT * FROM users');
  for (const user of users) {
    user.posts = await db.query(
      'SELECT * FROM posts WHERE user_id = ?',
      [user.id]
    );
  }
  return users;
}

// ✅ 使用 JOIN 或批量查询
async function getUsersWithPosts() {
  return await db.query(`
    SELECT u.*, p.*
    FROM users u
    LEFT JOIN posts p ON u.id = p.user_id
  `);
}

// ✅ 使用索引
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_posts_user_id ON posts(user_id);

// ✅ 使用缓存
import NodeCache from 'node-cache';

const cache = new NodeCache({ stdTTL: 600 }); // 10 分钟

async function getUserById(id: string): Promise<User> {
  const cacheKey = `user:${id}`;
  const cached = cache.get<User>(cacheKey);

  if (cached) {
    return cached;
  }

  const user = await db.query('SELECT * FROM users WHERE id = ?', [id]);
  cache.set(cacheKey, user);

  return user;
}

// ✅ Redis 缓存
import Redis from 'ioredis';

const redis = new Redis();

async function getUserById(id: string): Promise<User> {
  const cacheKey = `user:${id}`;
  const cached = await redis.get(cacheKey);

  if (cached) {
    return JSON.parse(cached);
  }

  const user = await db.query('SELECT * FROM users WHERE id = ?', [id]);
  await redis.setex(cacheKey, 3600, JSON.stringify(user)); // 1 小时

  return user;
}
```

### API 性能

```typescript
// ✅ 压缩响应
import compression from 'compression';

app.use(compression({
  level: 6,
  threshold: 1024,
}));

// ✅ 分页
app.get('/api/users', async (req, res) => {
  const { page = 1, size = 20 } = req.query;
  const offset = (page - 1) * size;

  const users = await db.query(
    'SELECT * FROM users LIMIT ? OFFSET ?',
    [size, offset]
  );

  const total = await db.query('SELECT COUNT(*) FROM users');

  res.json({
    data: users,
    pagination: {
      page: Number(page),
      size: Number(size),
      total: total[0].count,
    },
  });
});

// ✅ 字段选择
app.get('/api/users/:id', async (req, res) => {
  const { fields } = req.query;
  const selectedFields = fields ? fields.split(',') : ['*'];

  const user = await db.query(
    `SELECT ${selectedFields.join(',')} FROM users WHERE id = ?`,
    [req.params.id]
  );

  res.json({ data: user });
});

// ✅ ETag 缓存
import etag from 'etag';

app.get('/api/users/:id', async (req, res) => {
  const user = await getUserById(req.params.id);
  const etagValue = etag(JSON.stringify(user));

  res.set('ETag', etagValue);
  res.set('Cache-Control', 'private, max-age=3600');

  if (req.headers['if-none-match'] === etagValue) {
    return res.status(304).end();
  }

  res.json({ data: user });
});
```

### 构建优化

```typescript
// ✅ Webpack 优化
module.exports = {
  optimization: {
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          chunks: 'all',
        },
      },
    },
    minimize: true,
    minimizer: [
      new TerserPlugin({
        terserOptions: {
          compress: {
            drop_console: true,
          },
        },
      }),
    ],
  },
};

// ✅ Vite 优化
export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom'],
          router: ['react-router-dom'],
        },
      },
    },
    minify: 'terser',
    terserOptions: {
      compress: {
        drop_console: true,
      },
    },
  },
});
```

## 性能监控

```typescript
// ✅ Web Vitals 监控
import { onCLS, onFID, onLCP, onFCP, onTTFB } from 'web-vitals';

function sendToAnalytics(metric: Metric) {
  const body = JSON.stringify({
    name: metric.name,
    value: metric.value,
    id: metric.id,
    delta: metric.delta,
  });

  // 使用 Beacon API 发送
  if (navigator.sendBeacon) {
    navigator.sendBeacon('/api/analytics/vitals', body);
  }
}

onCLS(sendToAnalytics);
onFID(sendToAnalytics);
onLCP(sendToAnalytics);
onFCP(sendToAnalytics);
onTTFB(sendToAnalytics);

// ✅ 自定义性能标记
function measurePerformance(name: string, fn: () => void) {
  performance.mark(`${name}-start`);
  fn();
  performance.mark(`${name}-end`);

  performance.measure(name, `${name}-start`, `${name}-end`);

  const measure = performance.getEntriesByName(name)[0];
  console.log(`${name}: ${measure.duration}ms`);
}
```

## 性能预算

```json
// .performance-budget.json
{
  "budgets": [
    {
      "path": "/",
      "timings": [
        { "metric": "LCP", "budget": 2500 },
        { "metric": "FID", "budget": 100 },
        { "metric": "CLS", "budget": 0.1 }
      ],
      "resourceSizes": [
        { "resourceType": "script", "budget": 200 },
        { "resourceType": "style", "budget": 50 },
        { "resourceType": "image", "budget": 300 },
        { "resourceType": "total", "budget": 500 }
      ]
    }
  ]
}
```

## 常见陷阱

### ❌ 避免
```typescript
// ❌ 过早优化
// 在没有测量的情况下优化代码

// ❌ 在循环中访问 DOM
for (let i = 0; i < 1000; i++) {
  document.getElementById('list').innerHTML += `<li>${i}</li>`;
}

// ❌ 不必要的重渲染
function Parent() {
  const [count, setCount] = useState(0);
  return <ExpensiveChild onClick={() => setCount(count + 1)} />;
}

// ❌ 同步加载所有资源
import HeavyLibrary from 'heavy-library';
```

### ✅ 推荐
```typescript
// ✅ 先测量
const start = performance.now();
// ... 操作
const end = performance.now();
console.log(`耗时: ${end - start}ms`);

// ✅ 使用 DocumentFragment
const fragment = document.createDocumentFragment();
for (let i = 0; i < 1000; i++) {
  const li = document.createElement('li');
  li.textContent = String(i);
  fragment.appendChild(li);
}
document.getElementById('list').appendChild(fragment);

// ✅ 使用 useCallback
function Parent() {
  const [count, setCount] = useState(0);
  const handleClick = useCallback(() => setCount(c => c + 1), []);
  return <ExpensiveChild onClick={handleClick} />;
}

// ✅ 懒加载
const HeavyLibrary = lazy(() => import('heavy-library'));
```

## 依赖推荐

- **监控**: Sentry Performance / New Relic / Datadog
- **分析**: Google Analytics / Mixpanel / Amplitude
- **构建**: Vite / Webpack / esbuild
- **图片优化**: sharp / imagemin / squoosh
- **CDN**: Cloudflare / Fastly / AWS CloudFront

## 项目特定规则

> 💡 在下方添加你的项目特定规则

```markdown
## 项目配置

- 性能预算：
- 监控工具：
- CDN：
- 目标指标：
```
