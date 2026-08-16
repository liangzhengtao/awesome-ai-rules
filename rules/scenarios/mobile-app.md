# Mobile App AI 编程规则
> Last updated: 2026 | Targets: React Native 0.74+ / Flutter 3.22+

## 核心原则

- 优先考虑用户体验和性能
- 遵循平台设计规范（iOS Human Interface / Material Design）
- 使用跨平台框架时保持原生体验
- 合理管理应用生命周期
- 处理网络状态和离线场景

## 代码风格

### React Native 项目结构
```
src/
├── app/                    # App 入口和配置
│   ├── navigation/         # 导航配置
│   └── providers/          # Context Providers
├── components/             # 共享组件
│   ├── ui/                 # UI 基础组件
│   └── common/             # 业务通用组件
├── screens/                # 页面
│   ├── Home/
│   │   ├── HomeScreen.tsx
│   │   ├── hooks.ts
│   │   └── components/
├── hooks/                  # 自定义 Hooks
├── services/               # API 服务
├── store/                  # 状态管理
├── utils/                  # 工具函数
├── types/                  # TypeScript 类型
└── assets/                 # 静态资源
```

### Flutter 项目结构
```
lib/
├── app/                    # App 配置
│   ├── routes/             # 路由配置
│   └── themes/             # 主题配置
├── features/               # 功能模块
│   ├── auth/
│   │   ├── data/           # 数据层
│   │   ├── domain/         # 领域层
│   │   └── presentation/   # 表现层
├── shared/                 # 共享代码
│   ├── widgets/            # 共享组件
│   ├── utils/              # 工具函数
│   └── constants/          # 常量
└── main.dart
```

## 最佳实践

### React Native

```typescript
// ✅ 使用 React Native 优化组件
import { FlashList } from '@shopify/flash-list';
import { MotiView } from 'moti';

// ✅ 使用 FlashList 替代 FlatList
<FlashList
  data={items}
  renderItem={({ item }) => <ItemCard item={item} />}
  estimatedItemSize={100}
  onEndReached={loadMore}
  onEndReachedThreshold={0.5}
/>

// ✅ 使用 Moti 实现流畅动画
<MotiView
  from={{ opacity: 0, translateY: 20 }}
  animate={{ opacity: 1, translateY: 0 }}
  transition={{ type: 'timing', duration: 300 }}
>
  <Text>Hello</Text>
</MotiView>

// ✅ 自定义 Hook 处理网络状态
function useNetworkStatus() {
  const [isConnected, setIsConnected] = useState(true);

  useEffect(() => {
    const unsubscribe = NetInfo.addEventListener(state => {
      setIsConnected(state.isConnected ?? false);
    });

    return unsubscribe;
  }, []);

  return isConnected;
}

// ✅ 使用 React Query 管理服务端状态
function useUser(userId: string) {
  return useQuery({
    queryKey: ['user', userId],
    queryFn: () => api.getUser(userId),
    staleTime: 5 * 60 * 1000, // 5 分钟
    cacheTime: 30 * 60 * 1000, // 30 分钟
  });
}
```

### Flutter

```dart
// ✅ 使用 BLoC 模式
// events.dart
abstract class UserEvent {}

class LoadUser extends UserEvent {
  final String userId;
  LoadUser(this.userId);
}

// state.dart
abstract class UserState {}

class UserLoading extends UserState {}

class UserLoaded extends UserState {
  final User user;
  UserLoaded(this.user);
}

class UserError extends UserState {
  final String message;
  UserError(this.message);
}

// bloc.dart
class UserBloc extends Bloc<UserEvent, UserState> {
  final UserRepository repository;

  UserBloc(this.repository) : super(UserInitial()) {
    on<LoadUser>(_onLoadUser);
  }

  Future<void> _onLoadUser(LoadUser event, Emitter<UserState> emit) async {
    emit(UserLoading());

    try {
      final user = await repository.getUser(event.userId);
      emit(UserLoaded(user));
    } catch (e) {
      emit(UserError(e.toString()));
    }
  }
}

// ✅ 使用 Riverpod 进行状态管理
final userProvider = FutureProvider.family<User, String>((ref, userId) async {
  final repository = ref.watch(userRepositoryProvider);
  return repository.getUser(userId);
});

// ✅ 在 Widget 中使用
class UserScreen extends ConsumerWidget {
  final String userId;

  const UserScreen({required this.userId});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final userAsync = ref.watch(userProvider(userId));

    return userAsync.when(
      data: (user) => UserView(user: user),
      loading: () => const CircularProgressIndicator(),
      error: (error, stack) => ErrorView(error: error),
    );
  }
}
```

### 性能优化

```typescript
// ✅ React Native - 图片优化
import FastImage from 'react-native-fast-image';

<FastImage
  source={{ uri: imageUrl, priority: FastImage.priority.normal }}
  style={styles.image}
  resizeMode={FastImage.resizeMode.cover}
/>

// ✅ React Native - 列表优化
const ItemCard = React.memo(({ item }: { item: Item }) => {
  return (
    <View style={styles.card}>
      <Text>{item.title}</Text>
    </View>
  );
});

// ✅ React Native - 使用 useMemo 和 useCallback
function ItemList({ items, onSelect }: Props) {
  const sortedItems = useMemo(() => {
    return [...items].sort((a, b) => a.name.localeCompare(b.name));
  }, [items]);

  const handleSelect = useCallback((id: string) => {
    onSelect(id);
  }, [onSelect]);

  return (
    <FlashList
      data={sortedItems}
      renderItem={({ item }) => (
        <ItemCard item={item} onSelect={handleSelect} />
      )}
    />
  );
}
```

```dart
// ✅ Flutter - 使用 const 构造函数
class MyWidget extends StatelessWidget {
  const MyWidget({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return const Column(
      children: [
        Text('Hello'),  // const Text
        Icon(Icons.star), // const Icon
      ],
    );
  }
}

// ✅ Flutter - 避免不必要的重建
class OptimizedWidget extends StatelessWidget {
  const OptimizedWidget({required this.data});

  final Data data;

  @override
  Widget build(BuildContext context) {
    return RepaintBoundary(
      child: CustomPaint(
        painter: MyPainter(data),
      ),
    );
  }
}
```

### 离线支持

```typescript
// ✅ React Native - 使用 MMKV 存储
import { MMKV } from 'react-native-mmkv';

const storage = new MMKV();

// 存储
storage.set('user', JSON.stringify(user));

// 读取
const userData = storage.getString('user');
const user = userData ? JSON.parse(userData) : null;

// ✅ 使用 WatermelonDB 进行本地数据库
import { Database, Model } from '@nozbe/watermelondb';
import { field, relation } from '@nozbe/watermelondb/decorators';

class User extends Model {
  static table = 'users';

  @field('username') username!: string;
  @field('email') email!: string;
  @field('avatar_url') avatarUrl?: string;
}

// ✅ 离线同步策略
function useOfflineSync() {
  const queryClient = useQueryClient();

  useEffect(() => {
    const unsubscribe = NetInfo.addEventListener(state => {
      if (state.isConnected) {
        // 上线时同步数据
        queryClient.invalidateQueries();
        syncPendingChanges();
      }
    });

    return unsubscribe;
  }, []);
}
```

### 推送通知

```typescript
// ✅ React Native - Firebase 推送
import messaging from '@react-native-firebase/messaging';

async function requestPermission() {
  const authStatus = await messaging().requestPermission();
  return (
    authStatus === messaging.AuthorizationStatus.AUTHORIZED ||
    authStatus === messaging.AuthorizationStatus.PROVISIONAL
  );
}

async function getFCMToken() {
  const token = await messaging().getToken();
  return token;
}

// 监听通知
messaging().onMessage(async remoteMessage => {
  console.log('Foreground message:', remoteMessage);
});

messaging().onNotificationOpenedApp(remoteMessage => {
  console.log('Background message:', remoteMessage);
});
```

```dart
// ✅ Flutter - Firebase 推送
import 'package:firebase_messaging/firebase_messaging.dart';

class NotificationService {
  final FirebaseMessaging _fcm = FirebaseMessaging.instance;

  Future<void> initialize() async {
    // 请求权限
    NotificationSettings settings = await _fcm.requestPermission();

    // 获取 token
    String? token = await _fcm.getToken();
    print('FCM Token: $token');

    // 前台消息
    FirebaseMessaging.onMessage.listen((RemoteMessage message) {
      print('Foreground message: ${message.notification?.title}');
    });

    // 后台消息
    FirebaseMessaging.onMessageOpenedApp.listen((RemoteMessage message) {
      print('Background message: ${message.notification?.title}');
    });
  }
}
```

### 导航

```typescript
// ✅ React Native - React Navigation
import { createNativeStackNavigator } from '@react-navigation/native-stack';
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';

const Stack = createNativeStackNavigator<RootStackParamList>();
const Tab = createBottomTabNavigator<RootTabParamList>();

function HomeTabs() {
  return (
    <Tab.Navigator>
      <Tab.Screen name="Home" component={HomeScreen} />
      <Tab.Screen name="Search" component={SearchScreen} />
      <Tab.Screen name="Profile" component={ProfileScreen} />
    </Tab.Navigator>
  );
}

function App() {
  return (
    <NavigationContainer>
      <Stack.Navigator>
        <Stack.Screen name="Main" component={HomeTabs} options={{ headerShown: false }} />
        <Stack.Screen name="UserDetail" component={UserDetailScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
}
```

```dart
// ✅ Flutter - GoRouter
import 'package:go_router/go_router.dart';

final router = GoRouter(
  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => const HomeScreen(),
      routes: [
        GoRoute(
          path: 'user/:id',
          builder: (context, state) {
            final userId = state.pathParameters['id']!;
            return UserScreen(userId: userId);
          },
        ),
      ],
    ),
  ],
);
```

## 测试

```typescript
// ✅ React Native 组件测试
import { render, fireEvent, waitFor } from '@testing-library/react-native';

describe('UserScreen', () => {
  it('displays user data', async () => {
    const { getByText } = render(<UserScreen userId="1" />);

    await waitFor(() => {
      expect(getByText('John Doe')).toBeTruthy();
    });
  });

  it('handles button press', async () => {
    const onPress = jest.fn();
    const { getByText } = render(<Button onPress={onPress} title="Click me" />);

    fireEvent.press(getByText('Click me'));
    expect(onPress).toHaveBeenCalled();
  });
});
```

```dart
// ✅ Flutter Widget 测试
void main() {
  testWidgets('UserScreen displays user data', (WidgetTester tester) async {
    await tester.pumpWidget(
      MaterialApp(
        home: UserScreen(userId: '1'),
      ),
    );

    await tester.pumpAndSettle();

    expect(find.text('John Doe'), findsOneWidget);
  });
}
```

## 常见陷阱

### ❌ 避免
```typescript
// ❌ 在渲染中创建新对象
<View style={{ padding: 10 }}>  // 每次渲染都创建新对象
  <Text>Hello</Text>
</View>

// ❌ 不必要的重渲染
function Parent() {
  const [count, setCount] = useState(0);
  return <Child onPress={() => setCount(count + 1)} />;
}
```

### ✅ 推荐
```typescript
// ✅ 使用 StyleSheet
const styles = StyleSheet.create({
  container: { padding: 10 },
});

<View style={styles.container}>
  <Text>Hello</Text>
</View>

// ✅ 使用 useCallback
function Parent() {
  const [count, setCount] = useState(0);
  const handlePress = useCallback(() => setCount(c => c + 1), []);
  return <Child onPress={handlePress} />;
}
```

## 依赖推荐

### React Native
- **导航**: React Navigation
- **状态管理**: Zustand / Jotai
- **数据获取**: React Query
- **存储**: MMKV
- **UI 组件**: React Native Paper / NativeBase
- **动画**: React Native Reanimated / Moti

### Flutter
- **状态管理**: Riverpod / BLoC / GetX
- **路由**: GoRouter
- **网络**: Dio
- **存储**: Hive / Isar
- **UI 组件**: Material 3 / Flutter ScreenUtil

## 项目特定规则

> 💡 在下方添加你的项目特定规则

```markdown
## 项目配置

- 项目名称：
- 框架：React Native / Flutter
- 最低支持版本：iOS 14+ / Android 8+
- 状态管理：
- 导航方案：
```
