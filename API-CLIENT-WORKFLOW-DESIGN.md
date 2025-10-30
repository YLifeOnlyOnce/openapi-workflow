# API Client 工作流设计方案

## 一、整体架构概览

本方案旨在为多团队提供统一的 API Client 解决方案，基于 OpenAPI 规范自动生成前端 SDK，实现标准化的 API 调用和维护。

### 核心目标
- **标准化**：统一 API 调用方式和错误处理
- **自动化**：从 OpenAPI 规范自动生成 SDK
- **可复用**：多团队共享基础能力
- **可扩展**：支持团队级定制化需求

---

## 二、工作流环节拆解

### 2.1 环节一：OpenAPI 规范管理层
**职责：定义和维护 API 契约**

#### 输入
- 后端服务的 API 定义文档
- 各微服务的 OpenAPI Specification (3.0+)

#### 产出
- 标准化的 OpenAPI JSON/YAML 文件
- 版本化的 API 规范文档

#### 关键活动
1. **规范采集**
   - 从后端服务自动拉取 OpenAPI 规范
   - 或手动维护 API 规范文件
   - 支持多个微服务/API 域

2. **规范验证**
   - 校验 OpenAPI 规范格式正确性
   - 检查必要字段完整性
   - 验证数据模型一致性

3. **版本管理**
   - 按版本号管理不同版本的规范
   - 支持规范变更历史追踪
   - 记录 Breaking Changes

#### 技术选型建议
- 存储：Git 仓库 / Artifactory / Nexus
- 验证工具：`@apidevtools/swagger-parser`, `openapi-validator`
- 版本策略：语义化版本 (Semantic Versioning)

---

### 2.2 环节二：代码生成层
**职责：将 OpenAPI 规范转换为 TypeScript 代码**

#### 输入
- OpenAPI 规范文件
- 代码生成配置（模板、命名规则等）

#### 产出
- TypeScript 类型定义 (interfaces, types)
- API 请求函数
- 数据模型类

#### 关键活动
1. **类型生成**
   ```
   OpenAPI Schema → TypeScript Interface/Type
   ```
   - Request Body Types
   - Response Types
   - Path Parameters, Query Parameters Types
   - Enum 类型

2. **API 函数生成**
   - 为每个 API endpoint 生成对应的函数
   - 函数签名包含类型信息
   - 自动处理路径参数、查询参数、请求体

3. **文档注释生成**
   - 从 OpenAPI description 生成 JSDoc
   - 保留参数说明和示例

#### 技术选型建议
- 生成工具：
  - `openapi-typescript` (仅类型)
  - `openapi-generator-cli` (完整 SDK)
  - `orval` (React Query 集成)
  - `swagger-typescript-api`

#### 配置示例
```yaml
# codegen.config.yaml
generator: openapi-typescript-codegen
output: ./generated
input: ./specs/api-v1.yaml
options:
  clientName: ApiClient
  useOptions: true
  useUnionTypes: true
  exportCore: true
  exportServices: true
  exportModels: true
```

---

### 2.3 环节三：基础请求层 (Core Layer)
**职责：提供统一的 HTTP 请求能力和通用逻辑**

#### 输入
- Axios 实例配置
- 全局拦截器配置
- 公共业务逻辑

#### 产出
- 封装好的 Axios 实例
- 请求/响应拦截器
- 错误处理器

#### 关键活动
1. **Axios 实例管理**
   ```typescript
   // 伪代码示例
   class HttpClient {
     - baseURL
     - timeout
     - headers
     - interceptors
   }
   ```

2. **请求拦截器**
   - 添加认证 Token
   - 添加公共 Header (如 tenant-id, trace-id)
   - 请求参数预处理
   - 请求日志记录

3. **响应拦截器**
   - 统一响应格式转换
   - 业务错误码处理
   - 响应日志记录
   - 数据解包 (unwrap)

4. **错误处理**
   - HTTP 状态码错误处理 (401, 403, 404, 500...)
   - 业务错误码映射
   - 网络错误处理
   - 超时处理
   - 错误重试逻辑

5. **通用功能**
   - 请求取消 (CancelToken)
   - 请求去重
   - 请求缓存
   - 请求队列管理
   - 上传/下载进度

#### 设计模式
- **单例模式**：全局共享 Axios 实例
- **责任链模式**：拦截器链式处理
- **策略模式**：不同错误处理策略

---

### 2.4 环节四：业务适配层 (Adapter Layer)
**职责：桥接生成代码和业务需求，提供定制化能力**

#### 输入
- 生成的 API 函数
- 业务特定需求

#### 产出
- 适配后的 API 调用方法
- 业务级别的数据转换
- 团队级别的定制功能

#### 关键活动
1. **数据转换**
   - 请求数据格式转换（前端格式 → 后端格式）
   - 响应数据格式转换（后端格式 → 前端格式）
   - 日期格式、枚举值、空值处理

2. **业务逻辑注入**
   - 特定 API 的前置/后置处理
   - 业务级别的缓存策略
   - 特定接口的重试策略
   - 数据校验和规范化

3. **团队定制**
   - 不同团队可扩展自己的 Adapter
   - 支持 Hooks/Plugins 机制
   - 允许覆盖默认行为

#### 设计模式
- **适配器模式**：转换接口格式
- **装饰器模式**：增强 API 功能
- **插件模式**：支持扩展

---

### 2.5 环节五：SDK 封装层 (SDK Package)
**职责：将所有层整合为可发布的 NPM 包**

#### 输入
- 生成的代码
- 核心请求层
- 适配层
- 配置文件

#### 产出
- 编译后的 NPM 包
- TypeScript 类型声明文件 (.d.ts)
- 使用文档

#### 关键活动
1. **代码打包**
   - 使用 Rollup/Webpack/Vite 打包
   - 生成多种格式：ESM, CommonJS, UMD
   - Tree-shaking 优化
   - 代码压缩

2. **类型声明**
   - 生成完整的 .d.ts 文件
   - 确保 TypeScript 类型提示正常

3. **版本发布**
   - 遵循语义化版本
   - 生成 Changelog
   - 发布到私有 NPM Registry

4. **文档生成**
   - API 参考文档
   - 使用示例
   - 迁移指南（版本升级）

#### 打包配置示例
```json
{
  "name": "@company/api-client",
  "version": "1.0.0",
  "main": "dist/index.cjs.js",
  "module": "dist/index.esm.js",
  "types": "dist/index.d.ts",
  "exports": {
    ".": {
      "import": "./dist/index.esm.js",
      "require": "./dist/index.cjs.js"
    }
  }
}
```

---

### 2.6 环节六：消费使用层 (Consumer Apps)
**职责：业务应用集成和使用 SDK**

#### 输入
- 发布的 NPM 包
- 业务应用代码

#### 产出
- 集成 API Client 的业务应用

#### 关键活动
1. **安装依赖**
   ```bash
   npm install @company/api-client
   ```

2. **初始化配置**
   ```typescript
   // 伪代码
   import { createApiClient } from '@company/api-client'
   
   const client = createApiClient({
     baseURL: 'https://api.example.com',
     token: () => getAuthToken(),
     onError: (error) => handleError(error)
   })
   ```

3. **调用 API**
   ```typescript
   // 伪代码
   const { data } = await client.user.getProfile({ userId: '123' })
   ```

4. **框架集成**
   - React: 配合 React Query / SWR 使用
   - Vue: 配合 Pinia / Composition API
   - 提供对应的 Hooks / Composables

---

## 三、交互关系图

```
┌─────────────────────────────────────────────────────────────────┐
│                    API Client 工作流全景                          │
└─────────────────────────────────────────────────────────────────┘

        后端服务                        前端团队
           │                              ▲
           │                              │
           ▼                              │
┌──────────────────────┐                 │
│  1. OpenAPI 规范管理  │                 │
│                      │                 │
│  • 采集规范          │                 │
│  • 验证格式          │                 │
│  • 版本管理          │                 │
└──────────┬───────────┘                 │
           │                              │
           │ [OpenAPI Spec]              │
           ▼                              │
┌──────────────────────┐                 │
│  2. 代码生成层        │                 │
│                      │                 │
│  • 类型生成          │                 │
│  • API 函数生成      │                 │
│  • 文档注释生成      │                 │
└──────────┬───────────┘                 │
           │                              │
           │ [Generated Code]            │
           ▼                              │
┌──────────────────────┐                 │
│  3. 基础请求层        │◄────────────────┤
│     (Core Layer)     │                 │
│                      │                 │
│  • Axios 实例        │                 │
│  • 拦截器            │                 │
│  • 错误处理          │                 │
│  • 通用功能          │                 │
└──────────┬───────────┘                 │
           │                              │
           │ [HTTP Client]               │
           ▼                              │
┌──────────────────────┐                 │
│  4. 业务适配层        │◄────────────────┤
│   (Adapter Layer)    │   团队定制扩展   │
│                      │                 │
│  • 数据转换          │                 │
│  • 业务逻辑注入      │                 │
│  • 团队定制          │                 │
└──────────┬───────────┘                 │
           │                              │
           │ [Adapted APIs]              │
           ▼                              │
┌──────────────────────┐                 │
│  5. SDK 封装层        │                 │
│    (Package)         │                 │
│                      │                 │
│  • 代码打包          │                 │
│  • 类型声明          │                 │
│  • 版本发布          │                 │
│  • 文档生成          │                 │
└──────────┬───────────┘                 │
           │                              │
           │ [NPM Package]               │
           ▼                              │
┌──────────────────────┐                 │
│  6. 消费使用层        │─────────────────┘
│   (Consumer Apps)    │
│                      │
│  • React App         │
│  • Vue App           │
│  • Angular App       │
└──────────────────────┘
```

---

## 四、数据流转示意

### 4.1 开发时流程（SDK 生成）

```
OpenAPI Spec → 代码生成器 → TS 类型 & API 函数
                               ↓
                         核心请求层封装
                               ↓
                         业务适配层处理
                               ↓
                         打包构建 & 发布
                               ↓
                         NPM Registry
```

### 4.2 运行时流程（API 调用）

```
业务代码调用
    ↓
SDK API 函数
    ↓
业务适配层（数据转换/逻辑注入）
    ↓
基础请求层（添加 Token/Header）
    ↓
Axios 请求拦截器
    ↓
HTTP 请求 → 后端服务
    ↓
HTTP 响应
    ↓
Axios 响应拦截器
    ↓
基础请求层（错误处理/格式转换）
    ↓
业务适配层（数据转换）
    ↓
返回给业务代码
```

---

## 五、关键设计决策

### 5.1 分层设计原则

**为什么要分层？**
- **职责分离**：每一层关注点不同，降低耦合
- **可替换性**：某一层的实现可以独立替换（如更换生成工具）
- **可扩展性**：在适配层提供扩展点，不影响核心层
- **可测试性**：每一层可以独立测试

### 5.2 生成代码 vs 手写代码

**哪些应该生成？**
- ✅ TypeScript 类型定义（完全自动生成）
- ✅ 基础 API 函数（自动生成）
- ✅ 数据模型类（自动生成）

**哪些应该手写？**
- ✅ 核心请求层（手写，高度可控）
- ✅ 拦截器逻辑（手写，业务相关）
- ✅ 错误处理策略（手写，业务相关）
- ✅ 业务适配逻辑（手写，团队定制）

### 5.3 单包 vs 多包策略

**方案 A：单一 SDK 包**
```
@company/api-client
  - 包含所有服务的 API
  - 统一版本管理
```
- 优点：简单，统一升级
- 缺点：体积大，全量更新

**方案 B：按服务拆分包**
```
@company/api-client-core    (核心层)
@company/api-client-user    (用户服务)
@company/api-client-order   (订单服务)
@company/api-client-payment (支付服务)
```
- 优点：按需引入，独立升级
- 缺点：版本管理复杂

**建议**：
- 初期使用单包策略，快速迭代
- 服务增多后，逐步拆分为多包
- Core 层始终独立，其他服务包依赖 Core

### 5.4 类型安全保证

**从 OpenAPI 到 TypeScript 的完整类型链**
```
OpenAPI Schema
    ↓ (代码生成)
TS Interface/Type
    ↓ (函数参数)
API 函数签名
    ↓ (运行时)
Axios 请求/响应类型
    ↓ (业务使用)
组件/逻辑中的类型推导
```

**关键点**：
- 所有 API 函数必须有完整的类型签名
- 使用泛型确保请求和响应类型匹配
- 避免使用 `any`，必要时使用 `unknown`

---

## 六、扩展能力设计

### 6.1 插件机制

允许团队扩展 SDK 功能，而不修改核心代码。

```typescript
// 伪代码示例
interface Plugin {
  name: string
  onRequest?: (config) => config
  onResponse?: (response) => response
  onError?: (error) => void
}

// 使用
apiClient.use(customPlugin)
```

### 6.2 中间件模式

类似 Express 中间件，链式处理请求。

```typescript
// 伪代码
apiClient.use(async (ctx, next) => {
  // 请求前处理
  console.log('Before:', ctx.config)
  
  await next()
  
  // 响应后处理
  console.log('After:', ctx.response)
})
```

### 6.3 配置继承

支持多级配置，优先级从高到低：
1. 单次请求配置
2. API 级别配置
3. 实例级别配置
4. 全局默认配置

---

## 七、质量保障

### 7.1 自动化测试

**单元测试**
- 核心请求层的拦截器逻辑
- 数据转换函数
- 错误处理器

**集成测试**
- Mock 服务器测试 API 调用
- 测试不同错误场景
- 测试拦截器链

**类型测试**
- 使用 `tsd` 或 `expect-type` 测试类型推导
- 确保 API 函数类型正确

### 7.2 规范校验

**OpenAPI 规范校验**
- 在生成代码前校验规范合法性
- 检测 Breaking Changes

**代码质量**
- ESLint 代码规范
- Prettier 格式化
- TypeScript 严格模式

### 7.3 文档完备性

**开发文档**
- 快速开始指南
- API 参考文档
- 架构设计文档（本文档）
- 贡献指南

**使用文档**
- 安装和配置
- 常见场景示例
- FAQ
- 最佳实践

---

## 八、持续集成/持续部署

### 8.1 CI/CD 流程

```
1. OpenAPI 规范变更
    ↓
2. 触发 CI Pipeline
    ↓
3. 校验规范 → 生成代码 → 运行测试
    ↓
4. 构建 NPM 包
    ↓
5. 发布到 Registry（版本标记：beta/latest）
    ↓
6. 通知相关团队
```

### 8.2 版本策略

**语义化版本**
- **Major (X.0.0)**：Breaking Changes（API 签名变更、删除）
- **Minor (0.X.0)**：新增功能（新 API、新参数，向后兼容）
- **Patch (0.0.X)**：Bug 修复、性能优化

**发布标签**
- `latest`：稳定版本
- `beta`：测试版本
- `alpha`：开发版本
- `legacy`：旧版本维护

---

## 九、监控和日志

### 9.1 请求监控

**关键指标**
- API 调用成功率
- API 响应时间
- 错误分布（按错误码）
- 各 API 的调用频次

**实现方式**
- 在拦截器中埋点
- 上报到监控平台（如 Sentry, DataDog）
- 生成可视化报表

### 9.2 日志记录

**日志级别**
- **ERROR**：请求失败、业务错误
- **WARN**：重试、降级
- **INFO**：请求开始、响应返回
- **DEBUG**：详细的请求参数、响应数据

**日志内容**
- 时间戳
- 请求 ID (trace-id)
- API 路径
- 请求参数
- 响应数据（脱敏）
- 错误信息

---

## 十、安全考虑

### 10.1 敏感信息保护

- Token 不应硬编码，从配置或回调函数获取
- 请求日志中脱敏处理（密码、身份证号等）
- 避免在 URL 中传递敏感参数

### 10.2 请求安全

- 支持请求签名
- HTTPS Only
- CSRF Token 处理
- 请求频率限制（客户端）

---

## 十一、落地实施路线图

### Phase 1：基础设施（1-2 周）
- [ ] 搭建 OpenAPI 规范管理仓库
- [ ] 选型并配置代码生成工具
- [ ] 实现核心请求层（Axios 封装）
- [ ] 编写基础测试

### Phase 2：自动化流程（1-2 周）
- [ ] 搭建 CI/CD Pipeline
- [ ] 实现自动化代码生成
- [ ] 配置 NPM Registry
- [ ] 实现自动发布

### Phase 3：功能增强（2-3 周）
- [ ] 实现业务适配层
- [ ] 开发插件机制
- [ ] 完善错误处理
- [ ] 添加监控和日志

### Phase 4：文档和推广（1 周）
- [ ] 编写完整文档
- [ ] 提供使用示例
- [ ] 在试点团队中试用
- [ ] 收集反馈并优化

### Phase 5：全面推广（持续）
- [ ] 推广到所有团队
- [ ] 定期维护和更新
- [ ] 版本迭代
- [ ] 社区建设

---

## 十二、总结

### 核心优势
1. **自动化**：从 OpenAPI 到 SDK 全流程自动化
2. **类型安全**：端到端的 TypeScript 类型保障
3. **标准化**：统一的 API 调用方式和错误处理
4. **可扩展**：插件和适配器支持团队定制
5. **高质量**：完善的测试、文档、监控

### 关键成功因素
- 后端团队提供标准的 OpenAPI 规范
- 自动化流程的搭建和维护
- 各团队的配合和反馈
- 持续的迭代和优化

### 潜在风险和应对
| 风险 | 影响 | 应对措施 |
|-----|------|---------|
| 后端 API 频繁变更 | SDK 频繁升级 | 制定 API 版本策略，向后兼容 |
| OpenAPI 规范不完整 | 生成代码质量差 | 规范校验，人工审核 |
| 多团队需求冲突 | 扩展困难 | 插件机制，适配层分离 |
| 维护成本高 | 难以持续 | 自动化 CI/CD，社区协作 |

---

## 附录

### A. 技术栈参考

**代码生成工具**
- openapi-typescript
- openapi-generator-cli
- swagger-typescript-api
- orval

**构建工具**
- Rollup (推荐用于库打包)
- Vite (开发体验好)
- Webpack

**测试工具**
- Vitest / Jest (单元测试)
- MSW (Mock Service Worker)
- tsd (类型测试)

**文档工具**
- TypeDoc (API 文档)
- VitePress / Docusaurus (文档站点)

### B. 参考资料
- [OpenAPI Specification](https://swagger.io/specification/)
- [Axios Documentation](https://axios-http.com/)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)
- [NPM Package Best Practices](https://docs.npmjs.com/packages-and-modules)

---

**文档版本**: 1.0  
**最后更新**: 2025-10-30  
**作者**: API Client Team
