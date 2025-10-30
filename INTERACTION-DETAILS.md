# API Client 工作流 - 详细交互说明

## 一、环节间的详细交互

### 1.1 OpenAPI 规范管理层 ↔ 代码生成层

**交互时机**: 每次需要更新 SDK 时（API 变更、新增接口等）

**交互方式**:
```
┌─────────────────────┐
│  OpenAPI 规范管理层  │
└──────────┬──────────┘
           │
           │ ① 提供规范文件
           │    - api-spec.yaml/json
           │    - 版本号信息
           │
           ▼
┌─────────────────────┐
│    代码生成层        │
│                     │
│  ② 读取并解析规范   │
│  ③ 执行生成脚本     │
└──────────┬──────────┘
           │
           │ ④ 输出生成结果
           │    - types/
           │    - services/
           │    - models/
           │
           ▼
        文件系统
```

**具体步骤**:
1. **规范获取**
   - 自动：CI 从后端仓库拉取最新 OpenAPI 文件
   - 手动：开发者下载并放入指定目录

2. **预处理**
   ```bash
   # 验证规范
   npm run validate:spec
   
   # 检测变更
   npm run detect:breaking-changes
   ```

3. **代码生成**
   ```bash
   # 生成命令
   openapi-generator-cli generate \
     -i ./specs/api-v1.yaml \
     -g typescript-axios \
     -o ./src/generated
   ```

4. **后处理**
   - 格式化生成的代码（Prettier）
   - 添加自定义注释头
   - 生成变更日志

---

### 1.2 代码生成层 ↔ 基础请求层

**交互时机**: 生成的 API 函数调用基础请求层发送 HTTP 请求

**交互方式**:
```
┌─────────────────────┐
│   代码生成层输出     │
│  (Generated APIs)   │
└──────────┬──────────┘
           │
           │ 生成的 API 函数依赖基础请求实例
           │
           ▼
    export function getUser(userId: string) {
      return httpClient.get(`/users/${userId}`)
                       ▲
                       └─── 使用基础请求层的 httpClient
                       
┌─────────────────────┐
│    基础请求层        │
│  (HttpClient)       │
│                     │
│  • get()            │
│  • post()           │
│  • put()            │
│  • delete()         │
└─────────────────────┘
```

**集成模式**:

**模式 1: 依赖注入**
```typescript
// 生成器配置中指定基础客户端
// generated/services/UserService.ts
export class UserService {
  constructor(private httpClient: HttpClient) {}
  
  async getUser(userId: string) {
    return this.httpClient.get(`/users/${userId}`)
  }
}
```

**模式 2: 单例模式**
```typescript
// 全局共享一个 httpClient 实例
// generated/services/UserService.ts
import { httpClient } from '@core/httpClient'

export async function getUser(userId: string) {
  return httpClient.get(`/users/${userId}`)
}
```

**推荐: 模式 2 (单例)** 
- 简单直接，生成代码更清晰
- 配置统一，便于管理
- 符合前端常见模式

---

### 1.3 基础请求层 ↔ 业务适配层

**交互时机**: 每次 API 请求和响应时

**交互方式**:
```
业务代码调用
    ↓
┌─────────────────────┐
│   业务适配层         │
│                     │
│  transformRequest() │ ← 请求前转换
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│   基础请求层         │
│                     │
│  请求拦截器链        │ ← 添加 Token、Headers
│        ↓            │
│   Axios 请求        │
│        ↓            │
│   响应拦截器链       │ ← 处理错误、解包数据
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│   业务适配层         │
│                     │
│  transformResponse()│ ← 响应后转换
└──────────┬──────────┘
           │
           ▼
      返回给业务代码
```

**具体实现**:

```typescript
// 1. 基础请求层提供钩子
class HttpClient {
  // 请求前钩子
  private beforeRequestHooks: Array<(config) => config> = []
  
  // 响应后钩子
  private afterResponseHooks: Array<(data) => data> = []
  
  // 注册钩子
  addBeforeRequestHook(hook) {
    this.beforeRequestHooks.push(hook)
  }
}

// 2. 业务适配层注册逻辑
// adapters/userAdapter.ts
httpClient.addBeforeRequestHook((config) => {
  if (config.url.startsWith('/users')) {
    // 对用户相关 API 的特殊处理
    config.data = transformUserRequest(config.data)
  }
  return config
})
```

---

### 1.4 业务适配层 ↔ SDK 封装层

**交互时机**: 打包构建时

**交互方式**:
```
┌─────────────────────┐
│  Generated Code     │
│  Core Layer         │
│  Adapter Layer      │
└──────────┬──────────┘
           │
           │ 所有源代码
           │
           ▼
┌─────────────────────┐
│   SDK 封装层         │
│   (Build Process)   │
│                     │
│  ① Tree-shaking     │ ← 移除未使用代码
│  ② Bundling         │ ← 打包成单文件或多文件
│  ③ Minification     │ ← 压缩代码
│  ④ Type Generation  │ ← 生成 .d.ts
└──────────┬──────────┘
           │
           │ 构建产物
           │
           ▼
┌─────────────────────┐
│   NPM Package       │
│                     │
│  dist/              │
│  ├─ index.esm.js    │
│  ├─ index.cjs.js    │
│  └─ index.d.ts      │
└─────────────────────┘
```

**关键点**:
- **导出策略**: 在 `index.ts` 中统一导出需要暴露的 API
- **类型完整性**: 确保所有导出都有对应的类型声明
- **按需加载**: 支持 ES Modules 的 Tree-shaking

```typescript
// src/index.ts - SDK 入口文件
export { createApiClient } from './core/client'
export { HttpClient } from './core/httpClient'
export * from './generated/services'
export * from './generated/models'
export * from './adapters'
export type * from './types'
```

---

### 1.5 SDK 封装层 ↔ 消费使用层

**交互时机**: 业务应用开发和运行时

**交互方式**:
```
┌─────────────────────┐
│   NPM Registry      │
│  @company/api-client│
└──────────┬──────────┘
           │
           │ npm install
           │
           ▼
┌─────────────────────┐
│   消费使用层         │
│  (Business App)     │
│                     │
│  ① 安装依赖          │
│  ② 初始化配置        │
│  ③ 调用 API         │
│  ④ 处理响应         │
└─────────────────────┘
```

**使用示例**:

**Step 1: 安装**
```bash
npm install @company/api-client
```

**Step 2: 初始化**
```typescript
// src/api/index.ts
import { createApiClient } from '@company/api-client'
import { getAuthToken } from '@/utils/auth'

export const apiClient = createApiClient({
  baseURL: import.meta.env.VITE_API_BASE_URL,
  timeout: 10000,
  
  // 认证 Token 获取
  getToken: () => getAuthToken(),
  
  // 全局错误处理
  onError: (error) => {
    if (error.code === 401) {
      // 跳转登录
      router.push('/login')
    }
  },
  
  // 自定义适配器
  adapters: [myCustomAdapter]
})
```

**Step 3: 在组件中使用**
```typescript
// React 组件示例
import { apiClient } from '@/api'

function UserProfile() {
  const [user, setUser] = useState(null)
  
  useEffect(() => {
    const fetchUser = async () => {
      try {
        const data = await apiClient.user.getProfile({ userId: '123' })
        setUser(data)
      } catch (error) {
        console.error('Failed to fetch user:', error)
      }
    }
    
    fetchUser()
  }, [])
  
  return <div>{user?.name}</div>
}
```

---

## 二、关键数据流转详解

### 2.1 请求数据流转

```
用户代码
  |
  | const data = await apiClient.user.create({ name: 'Alice' })
  ↓
┌──────────────────────────────────────────────┐
│ 1. 生成的 API 函数                            │
│    function createUser(data: CreateUserDto)  │
└────────────────┬─────────────────────────────┘
                 │
                 | data = { name: 'Alice' }
                 ↓
┌──────────────────────────────────────────────┐
│ 2. 业务适配层 - 请求转换                      │
│    • 格式转换 (camelCase → snake_case)       │
│    • 添加业务字段                             │
│    • 数据校验                                 │
└────────────────┬─────────────────────────────┘
                 │
                 | data = { user_name: 'Alice', tenant_id: 'xxx' }
                 ↓
┌──────────────────────────────────────────────┐
│ 3. 基础请求层 - 请求拦截器                    │
│    • 添加 Authorization Header               │
│    • 添加 X-Request-ID                       │
│    • 序列化数据                               │
└────────────────┬─────────────────────────────┘
                 │
                 | HTTP Request
                 |   Headers: { Authorization: 'Bearer xxx', ... }
                 |   Body: '{"user_name":"Alice","tenant_id":"xxx"}'
                 ↓
              Axios
                 ↓
            后端服务
```

### 2.2 响应数据流转

```
后端服务
  |
  | HTTP Response
  | Status: 200
  | Body: '{"code":0,"data":{"id":1,"user_name":"Alice"},"message":"success"}'
  ↓
Axios
  ↓
┌──────────────────────────────────────────────┐
│ 1. 基础请求层 - 响应拦截器                    │
│    • 检查 HTTP 状态码                         │
│    • 解析 JSON                               │
│    • 检查业务错误码                           │
│    • 数据解包                                 │
└────────────────┬─────────────────────────────┘
                 │
                 | data = { id: 1, user_name: 'Alice' }
                 ↓
┌──────────────────────────────────────────────┐
│ 2. 业务适配层 - 响应转换                      │
│    • 格式转换 (snake_case → camelCase)       │
│    • 类型转换 (字符串日期 → Date 对象)        │
│    • 数据规范化                               │
└────────────────┬─────────────────────────────┘
                 │
                 | data = { id: 1, userName: 'Alice' }
                 ↓
┌──────────────────────────────────────────────┐
│ 3. 生成的 API 函数返回                        │
│    return data (with TypeScript type)        │
└────────────────┬─────────────────────────────┘
                 │
                 ↓
              用户代码
              (得到类型安全的数据)
```

---

## 三、配置流转和优先级

### 3.1 配置层级

```
┌─────────────────────────────────────────────┐
│  Level 1: 全局默认配置                       │
│  (SDK 内置默认值)                            │
│                                             │
│  baseURL: ''                                │
│  timeout: 30000                             │
│  headers: { 'Content-Type': 'application/json' }
└───────────────────┬─────────────────────────┘
                    │ 覆盖
                    ↓
┌─────────────────────────────────────────────┐
│  Level 2: 实例级配置                         │
│  (createApiClient 时传入)                    │
│                                             │
│  baseURL: 'https://api.example.com'         │
│  timeout: 10000                             │
│  getToken: () => localStorage.get('token')  │
└───────────────────┬─────────────────────────┘
                    │ 覆盖
                    ↓
┌─────────────────────────────────────────────┐
│  Level 3: API 级配置                         │
│  (特定 API 的配置)                           │
│                                             │
│  uploadFile: {                              │
│    timeout: 60000,  // 上传文件超时时间更长  │
│  }                                          │
└───────────────────┬─────────────────────────┘
                    │ 覆盖
                    ↓
┌─────────────────────────────────────────────┐
│  Level 4: 单次请求配置                       │
│  (调用 API 时传入的 options)                 │
│                                             │
│  apiClient.user.get({                       │
│    userId: '123',                           │
│    _options: { timeout: 5000 }  // 这次请求5秒超时
│  })                                         │
└─────────────────────────────────────────────┘
```

### 3.2 配置合并策略

```typescript
// 伪代码示例
function mergeConfig(...configs) {
  // 深度合并，后面的配置覆盖前面的
  return {
    ...defaultConfig,      // Level 1
    ...instanceConfig,     // Level 2
    ...apiConfig,          // Level 3
    ...requestConfig,      // Level 4
    
    // 特殊处理：headers 需要合并而不是替换
    headers: {
      ...defaultConfig.headers,
      ...instanceConfig.headers,
      ...apiConfig.headers,
      ...requestConfig.headers,
    }
  }
}
```

---

## 四、错误处理流程

### 4.1 错误类型分类

```
错误来源
  │
  ├─ 网络错误 (Network Error)
  │   └─ 无网络、DNS 解析失败、连接超时
  │
  ├─ HTTP 状态码错误 (HTTP Status Error)
  │   ├─ 4xx: 客户端错误 (401, 403, 404, 422...)
  │   └─ 5xx: 服务器错误 (500, 502, 503...)
  │
  ├─ 业务错误码 (Business Error)
  │   └─ HTTP 200, 但 response.code !== 0
  │       例: { code: 1001, message: '用户名已存在' }
  │
  └─ 运行时错误 (Runtime Error)
      └─ 数据解析失败、类型错误等
```

### 4.2 错误处理流程

```
异常发生
  ↓
┌──────────────────────────────────────────────┐
│ 1. Axios 错误捕获                             │
│    axios.interceptors.response.use(           │
│      onSuccess,                               │
│      onError  ← 捕获                          │
│    )                                         │
└────────────────┬─────────────────────────────┘
                 │
                 ↓
┌──────────────────────────────────────────────┐
│ 2. 基础请求层 - 错误分类和包装                │
│    if (error.response) {                     │
│      // HTTP 状态码错误                       │
│      throw new HttpError(...)                │
│    } else if (error.request) {               │
│      // 网络错误                              │
│      throw new NetworkError(...)             │
│    } else {                                  │
│      // 运行时错误                            │
│      throw new RuntimeError(...)             │
│    }                                         │
└────────────────┬─────────────────────────────┘
                 │
                 ↓
┌──────────────────────────────────────────────┐
│ 3. 业务适配层 - 错误转换                      │
│    • 将错误码映射为业务含义                   │
│    • 添加上下文信息                           │
│    • 决定是否重试                             │
└────────────────┬─────────────────────────────┘
                 │
                 ↓
┌──────────────────────────────────────────────┐
│ 4. 全局错误处理器                             │
│    onError(error)                            │
│    • 401 → 跳转登录                          │
│    • 403 → 提示无权限                        │
│    • 500 → 显示系统错误                      │
│    • 上报错误日志                             │
└────────────────┬─────────────────────────────┘
                 │
                 ↓
              抛给用户代码
              (如果用户设置了 try-catch)
```

---

## 五、扩展机制详解

### 5.1 插件系统

**插件生命周期**:
```
SDK 初始化
  ↓
注册插件
  │
  ├─ plugin.install(apiClient)
  │   └─ 插件初始化，可以：
  │       • 添加拦截器
  │       • 扩展 API 方法
  │       • 注册全局处理器
  │
  └─ 生命周期钩子
      │
      ├─ onRequest(config)     ← 每次请求前
      ├─ onResponse(response)  ← 每次响应后
      ├─ onError(error)        ← 错误发生时
      └─ onDestroy()           ← SDK 销毁时
```

**插件示例**:
```typescript
// 日志插件
const loggingPlugin = {
  name: 'logging',
  
  install(apiClient) {
    console.log('Logging plugin installed')
  },
  
  onRequest(config) {
    console.log('[Request]', config.method, config.url)
    return config
  },
  
  onResponse(response) {
    console.log('[Response]', response.status, response.data)
    return response
  },
  
  onError(error) {
    console.error('[Error]', error.message)
  }
}

// 使用
apiClient.use(loggingPlugin)
```

### 5.2 适配器扩展

**团队自定义适配器**:
```typescript
// 团队 A 的适配器
export const teamAAdapter = {
  // 请求转换
  transformRequest(data, config) {
    // 团队 A 的特殊需求
    return {
      ...data,
      teamId: 'team-a',
      customField: 'value'
    }
  },
  
  // 响应转换
  transformResponse(data, config) {
    // 团队 A 的数据处理
    return normalizeData(data)
  },
  
  // 错误处理
  handleError(error) {
    // 团队 A 的错误处理逻辑
    if (error.code === 'TEAM_A_SPECIFIC_ERROR') {
      // 特殊处理
    }
    throw error
  }
}

// 注册
apiClient.addAdapter(teamAAdapter)
```

---

## 六、版本升级流程

### 6.1 版本发布流程

```
代码变更提交
  ↓
CI Pipeline 触发
  ↓
┌──────────────────────────────────────────────┐
│ 1. 自动化测试                                 │
│    • 单元测试                                 │
│    • 集成测试                                 │
│    • 类型检查                                 │
└────────────────┬─────────────────────────────┘
                 │ 通过
                 ↓
┌──────────────────────────────────────────────┐
│ 2. 版本号决策                                 │
│    • 检测 Breaking Changes → Major 升级       │
│    • 检测新功能 → Minor 升级                  │
│    • Bug 修复 → Patch 升级                   │
└────────────────┬─────────────────────────────┘
                 │
                 ↓
┌──────────────────────────────────────────────┐
│ 3. 构建和打包                                 │
│    • npm run build                           │
│    • 生成 Changelog                          │
│    • 打 Git Tag                              │
└────────────────┬─────────────────────────────┘
                 │
                 ↓
┌──────────────────────────────────────────────┐
│ 4. 发布到 NPM Registry                        │
│    • npm publish --tag beta (测试版)          │
│    • 测试通过后                                │
│    • npm dist-tag add @company/api-client@x.y.z latest
└────────────────┬─────────────────────────────┘
                 │
                 ↓
┌──────────────────────────────────────────────┐
│ 5. 通知用户                                   │
│    • 发送邮件/消息通知                        │
│    • 更新文档站点                             │
│    • 在内部论坛发布公告                       │
└──────────────────────────────────────────────┘
```

### 6.2 用户升级流程

```
收到升级通知
  ↓
查看 Changelog
  │
  ├─ 无 Breaking Changes
  │   └─ 直接升级
  │       npm install @company/api-client@latest
  │
  └─ 有 Breaking Changes
      └─ 查看迁移指南
          ↓
      评估改动影响
          ↓
      修改代码适配
          ↓
      测试验证
          ↓
      发布上线
```

---

## 七、监控和反馈循环

### 7.1 数据采集

```
用户使用 SDK
  ↓
SDK 内部埋点
  │
  ├─ 请求指标
  │   • API 路径
  │   • 请求耗时
  │   • 请求结果 (成功/失败)
  │
  ├─ 错误信息
  │   • 错误类型
  │   • 错误堆栈
  │   • 用户环境信息
  │
  └─ 使用统计
      • 功能使用频次
      • SDK 版本分布
  ↓
上报到监控平台
  ↓
数据分析和可视化
  ↓
发现问题 → 优化迭代
```

### 7.2 反馈渠道

```
用户反馈
  │
  ├─ Issue 提交 (GitHub/GitLab)
  ├─ 内部论坛讨论
  ├─ 定期用户访谈
  └─ 问卷调查
  ↓
需求收集和优先级排序
  ↓
纳入开发计划
  ↓
实现并发布
  ↓
通知用户
```

---

## 八、总结

### 关键交互点
1. **OpenAPI → 代码**: 自动化生成，减少手动维护
2. **生成代码 → 基础层**: 依赖注入或单例模式
3. **基础层 ↔ 适配层**: 钩子机制，双向数据转换
4. **SDK → 用户**: NPM 包，配置驱动
5. **运行时 → 监控**: 埋点上报，持续优化

### 设计原则
- **分层清晰**: 每层职责单一，易于维护
- **可扩展性**: 插件和适配器支持定制
- **自动化**: 从生成到发布全流程自动化
- **类型安全**: 端到端 TypeScript 支持
- **可观测性**: 完善的日志和监控

### 成功关键
- 自动化流程的建立
- 完善的文档和示例
- 快速的问题响应
- 持续的迭代优化
