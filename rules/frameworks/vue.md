# Vue.js AI 编程规则
> Last updated: 2026 | Targets: Vue 3.4+ / Composition API / Pinia

## 核心原则

- 使用 Vue 3 + Composition API
- 使用 `<script setup>` 语法糖
- 优先使用 TypeScript
- 遵循 Vue 风格指南
- 合理使用响应式 API

## 代码风格

### 命名规范
- 组件使用 PascalCase：`UserProfile.vue`, `NavigationBar.vue`
- 组合式函数使用 `use` 前缀：`useAuth`, `useFetch`
- 工具函数使用 camelCase：`formatDate`, `validateEmail`
- 常量使用 UPPER_SNAKE_CASE：`API_BASE_URL`
- Props 使用 camelCase，事件使用 kebab-case

### 文件结构
```
src/
├── assets/               # 静态资源
├── components/           # 公共组件
│   ├── Button/
│   │   ├── Button.vue
│   │   ├── Button.spec.ts
│   │   └── index.ts
├── composables/          # 组合式函数
├── layouts/              # 布局组件
├── pages/                # 页面组件
├── plugins/              # 插件
├── router/               # 路由配置
├── stores/               # 状态管理
├── types/                # TypeScript 类型
├── utils/                # 工具函数
└── styles/               # 全局样式
```

## 最佳实践

### 组件设计

```vue
<script setup lang="ts">
// ✅ Props 和 Emits 类型定义
interface Props {
  variant?: 'primary' | 'secondary' | 'danger';
  size?: 'sm' | 'md' | 'lg';
  loading?: boolean;
  disabled?: boolean;
}

interface Emits {
  (e: 'click', event: MouseEvent): void;
  (e: 'update:value', value: string): void;
}

// 使用 withDefaults 设置默认值
const props = withDefaults(defineProps<Props>(), {
  variant: 'primary',
  size: 'md',
  loading: false,
  disabled: false,
});

const emit = defineEmits<Emits>();

// 计算属性
const classes = computed(() => [
  'btn',
  `btn-${props.variant}`,
  `btn-${props.size}`,
  {
    'btn-loading': props.loading,
    'btn-disabled': props.disabled,
  },
]);

// 方法
const handleClick = (event: MouseEvent) => {
  if (!props.disabled && !props.loading) {
    emit('click', event);
  }
};
</script>

<template>
  <button
    :class="classes"
    :disabled="disabled || loading"
    @click="handleClick"
  >
    <span v-if="loading" class="spinner" />
    <slot />
  </button>
</template>
```

### 组合式函数

```typescript
// composables/useApi.ts
import { ref, shallowRef } from 'vue';

interface UseApiOptions<T> {
  immediate?: boolean;
  onSuccess?: (data: T) => void;
  onError?: (error: Error) => void;
}

export function useApi<T>(
  url: string | (() => string),
  options: UseApiOptions<T> = {}
) {
  const data = shallowRef<T | null>(null);
  const loading = ref(false);
  const error = ref<Error | null>(null);

  const execute = async () => {
    loading.value = true;
    error.value = null;

    try {
      const urlValue = typeof url === 'function' ? url() : url;
      const response = await fetch(urlValue);

      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }

      data.value = await response.json();
      options.onSuccess?.(data.value as T);
    } catch (err) {
      error.value = err instanceof Error ? err : new Error('Unknown error');
      options.onError?.(error.value);
    } finally {
      loading.value = false;
    }
  };

  if (options.immediate !== false) {
    execute();
  }

  return {
    data: readonly(data),
    loading: readonly(loading),
    error: readonly(error),
    execute,
    refresh: execute,
  };
}
```

### Pinia 状态管理

```typescript
// stores/user.ts
import { defineStore } from 'pinia';

interface User {
  id: string;
  name: string;
  email: string;
}

export const useUserStore = defineStore('user', () => {
  // State
  const currentUser = ref<User | null>(null);
  const isAuthenticated = computed(() => !!currentUser.value);

  // Actions
  async function login(email: string, password: string) {
    try {
      const response = await api.login(email, password);
      currentUser.value = response.user;
      return { success: true };
    } catch (error) {
      return { success: false, error: 'Login failed' };
    }
  }

  async function logout() {
    await api.logout();
    currentUser.value = null;
  }

  function updateProfile(updates: Partial<User>) {
    if (currentUser.value) {
      Object.assign(currentUser.value, updates);
    }
  }

  return {
    currentUser,
    isAuthenticated,
    login,
    logout,
    updateProfile,
  };
});
```

### Vue Router

```typescript
// router/index.ts
import { createRouter, createWebHistory } from 'vue-router';

const router = createRouter({
  history: createWebHistory(),
  routes: [
    {
      path: '/',
      component: () => import('@/layouts/MainLayout.vue'),
      children: [
        {
          path: '',
          name: 'home',
          component: () => import('@/pages/HomePage.vue'),
        },
        {
          path: 'profile/:id',
          name: 'profile',
          component: () => import('@/pages/ProfilePage.vue'),
          props: true,
        },
      ],
    },
  ],
});

// 路由守卫
router.beforeEach(async (to, from) => {
  const authStore = useAuthStore();

  if (to.meta.requiresAuth && !authStore.isAuthenticated) {
    return { name: 'login', query: { redirect: to.fullPath } };
  }
});

export default router;
```

### 表单处理

```vue
<script setup lang="ts">
import { useForm, useField } from 'vee-validate';
import { z } from 'zod';
import { toTypedSchema } from '@vee-validate/zod';

// 定义验证 schema
const schema = toTypedSchema(
  z.object({
    email: z.string().email('Invalid email'),
    password: z.string().min(8, 'Password must be at least 8 characters'),
  })
);

const { handleSubmit, errors, isSubmitting } = useForm({
  validationSchema: schema,
});

const { value: email } = useField('email');
const { value: password } = useField('password');

const onSubmit = handleSubmit(async (values) => {
  await authStore.login(values.email, values.password);
});
</script>

<template>
  <form @submit="onSubmit">
    <input v-model="email" type="email" />
    <span v-if="errors.email">{{ errors.email }}</span>

    <input v-model="password" type="password" />
    <span v-if="errors.password">{{ errors.password }}</span>

    <button :disabled="isSubmitting">Login</button>
  </form>
</template>
```

## 性能优化

```vue
<script setup lang="ts">
// ✅ 使用 shallowRef 处理大型对象
const largeList = shallowRef<Data[]>([]);

// ✅ 使用 v-once 处理静态内容
// <p v-once>This will never change: {{ staticContent }}</p>

// ✅ 使用 v-memo 优化列表渲染
// <div v-for="item in list" :key="item.id" v-memo="[item.selected]">
//   <p>{{ item.name }}</p>
//   <p>{{ item.selected ? 'Selected' : 'Not selected' }}</p>
// </div>
</script>
```

## 常见陷阱

### ❌ 避免
```typescript
// ❌ 解构 props 会失去响应性
const { name, age } = defineProps<Props>();

// ❌ 在 setup 外使用响应式 API
const count = ref(0); // 如果在 <script> 而非 <script setup> 中

// ❌ 直接修改 props
props.name = 'new name';
```

### ✅ 推荐
```typescript
// ✅ 使用 toRefs 保持响应性
const { name, age } = toRefs(props);

// ✅ 使用 computed 处理派生状态
const fullName = computed(() => `${firstName.value} ${lastName.value}`);

// ✅ 使用 emit 通知父组件
emit('update:name', newName);
```

## 依赖推荐

- **构建工具**: Vite
- **路由**: Vue Router
- **状态管理**: Pinia
- **表单验证**: VeeValidate + Zod
- **UI 组件库**: Element Plus / Naive UI / Vuetify
- **CSS 方案**: UnoCSS / Tailwind CSS
- **测试**: Vitest + Vue Test Utils
- **代码规范**: ESLint + Prettier

## 项目特定规则

> 💡 在下方添加你的项目特定规则

```markdown
## 项目配置

- 项目名称：
- UI 框架：
- 状态管理方案：
- 包管理器：
```
