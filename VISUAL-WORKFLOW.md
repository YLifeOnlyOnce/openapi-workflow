# API Client 工作流 - 可视化图表

本文档使用 Mermaid 图表语法展示工作流，可以在支持 Mermaid 的 Markdown 查看器中渲染（如 GitHub、GitLab、Notion、Typora 等）。

---

## 1. 整体架构图

```mermaid
graph TB
    subgraph Backend["后端服务"]
        API1[用户服务 API]
        API2[订单服务 API]
        API3[支付服务 API]
    end
    
    subgraph Layer1["1️⃣ OpenAPI 规范管理层"]
        SPEC1[user-api.yaml]
        SPEC2[order-api.yaml]
        SPEC3[payment-api.yaml]
        VALIDATOR[规范验证器]
    end
    
    subgraph Layer2["2️⃣ 代码生成层"]
        CODEGEN[代码生成器]
        TYPES[TypeScript 类型]
        SERVICES[API 服务函数]
    end
    
    subgraph Layer3["3️⃣ 基础请求层"]
        HTTP[HttpClient]
        INTERCEPTOR_REQ[请求拦截器]
        INTERCEPTOR_RES[响应拦截器]
        ERROR_HANDLER[错误处理器]
    end
    
    subgraph Layer4["4️⃣ 业务适配层"]
        ADAPTER_A[团队 A 适配器]
        ADAPTER_B[团队 B 适配器]
        TRANSFORM[数据转换器]
    end
    
    subgraph Layer5["5️⃣ SDK 封装层"]
        BUILD[构建打包]
        NPM[NPM 包]
    end
    
    subgraph Layer6["6️⃣ 消费使用层"]
        APP1[React App]
        APP2[Vue App]
        APP3[Angular App]
    end
    
    API1 --> SPEC1
    API2 --> SPEC2
    API3 --> SPEC3
    
    SPEC1 --> VALIDATOR
    SPEC2 --> VALIDATOR
    SPEC3 --> VALIDATOR
    
    VALIDATOR --> CODEGEN
    CODEGEN --> TYPES
    CODEGEN --> SERVICES
    
    SERVICES --> HTTP
    HTTP --> INTERCEPTOR_REQ
    INTERCEPTOR_REQ --> INTERCEPTOR_RES
    INTERCEPTOR_RES --> ERROR_HANDLER
    
    ERROR_HANDLER --> ADAPTER_A
    ERROR_HANDLER --> ADAPTER_B
    ADAPTER_A --> TRANSFORM
    ADAPTER_B --> TRANSFORM
    
    TRANSFORM --> BUILD
    BUILD --> NPM
    
    NPM --> APP1
    NPM --> APP2
    NPM --> APP3
    
    style Layer1 fill:#e1f5ff
    style Layer2 fill:#fff4e1
    style Layer3 fill:#ffe1f5
    style Layer4 fill:#f5ffe1
    style Layer5 fill:#ffe1e1
    style Layer6 fill:#e1e1ff
```

---

## 2. 开发时序图（SDK 生成流程）

```mermaid
sequenceDiagram
    participant BE as 后端服务
    participant SPEC as OpenAPI 规范
    participant GEN as 代码生成器
    participant CORE as 核心请求层
    participant PKG as 打包工具
    participant REG as NPM Registry
    
    BE->>SPEC: 1. 导出 OpenAPI 规范
    Note over SPEC: 存储在 Git 仓库
    
    SPEC->>GEN: 2. 触发代码生成
    Note over GEN: CI/CD 自动触发
    
    GEN->>GEN: 3. 解析规范
    GEN->>GEN: 4. 生成类型定义
    GEN->>GEN: 5. 生成 API 函数
    
    GEN->>CORE: 6. 集成核心层
    Note over CORE: 注入 HttpClient
    
    CORE->>PKG: 7. 准备打包
    PKG->>PKG: 8. 构建 ESM/CJS
    PKG->>PKG: 9. 生成类型声明
    PKG->>PKG: 10. 压缩优化
    
    PKG->>REG: 11. 发布 NPM 包
    Note over REG: 版本号: x.y.z
```

---

## 3. 运行时序图（API 调用流程）

```mermaid
sequenceDiagram
    participant USER as 业务代码
    participant SDK as SDK (生成的 API)
    participant ADAPTER as 业务适配层
    participant CORE as 基础请求层
    participant AXIOS as Axios
    participant BE as 后端服务
    
    USER->>SDK: 1. 调用 API 函数<br/>apiClient.user.create(data)
    
    SDK->>ADAPTER: 2. 请求前适配
    Note over ADAPTER: 数据格式转换<br/>添加业务字段
    
    ADAPTER->>CORE: 3. 发起请求
    
    CORE->>CORE: 4. 请求拦截器
    Note over CORE: 添加 Token<br/>添加 Headers<br/>请求日志
    
    CORE->>AXIOS: 5. Axios 执行请求
    AXIOS->>BE: 6. HTTP 请求
    
    BE-->>AXIOS: 7. HTTP 响应
    AXIOS-->>CORE: 8. 响应返回
    
    CORE->>CORE: 9. 响应拦截器
    Note over CORE: 检查状态码<br/>解析数据<br/>错误处理
    
    CORE-->>ADAPTER: 10. 响应后适配
    Note over ADAPTER: 数据格式转换<br/>类型规范化
    
    ADAPTER-->>SDK: 11. 返回数据
    SDK-->>USER: 12. 类型安全的数据
    Note over USER: TypeScript 类型提示
```

---

## 4. 错误处理流程图

```mermaid
graph TD
    START[API 请求发起] --> REQUEST[发送 HTTP 请求]
    REQUEST --> CHECK_RESPONSE{响应是否成功?}
    
    CHECK_RESPONSE -->|网络错误| NETWORK_ERROR[NetworkError]
    CHECK_RESPONSE -->|HTTP 4xx/5xx| HTTP_ERROR[HttpStatusError]
    CHECK_RESPONSE -->|HTTP 200| CHECK_BIZ{业务码检查}
    
    CHECK_BIZ -->|code !== 0| BIZ_ERROR[BusinessError]
    CHECK_BIZ -->|code === 0| SUCCESS[提取数据]
    
    NETWORK_ERROR --> ERROR_CLASSIFY[错误分类]
    HTTP_ERROR --> ERROR_CLASSIFY
    BIZ_ERROR --> ERROR_CLASSIFY
    
    ERROR_CLASSIFY --> ERROR_TRANSFORM[业务适配层<br/>错误转换]
    ERROR_TRANSFORM --> CHECK_RETRY{是否需要重试?}
    
    CHECK_RETRY -->|是| RETRY_COUNT{重试次数<br/>是否超限?}
    CHECK_RETRY -->|否| GLOBAL_HANDLER
    
    RETRY_COUNT -->|未超限| WAIT[等待间隔]
    RETRY_COUNT -->|已超限| GLOBAL_HANDLER[全局错误处理器]
    
    WAIT --> REQUEST
    
    GLOBAL_HANDLER --> CHECK_ERROR_TYPE{错误类型判断}
    CHECK_ERROR_TYPE -->|401 未授权| REDIRECT_LOGIN[跳转登录页]
    CHECK_ERROR_TYPE -->|403 无权限| SHOW_PERMISSION[显示权限提示]
    CHECK_ERROR_TYPE -->|500 服务器错误| SHOW_ERROR[显示错误提示]
    CHECK_ERROR_TYPE -->|其他| THROW[抛给用户代码]
    
    REDIRECT_LOGIN --> LOG[记录错误日志]
    SHOW_PERMISSION --> LOG
    SHOW_ERROR --> LOG
    THROW --> LOG
    
    LOG --> REPORT[上报监控平台]
    REPORT --> END[流程结束]
    
    SUCCESS --> DATA_TRANSFORM[数据转换]
    DATA_TRANSFORM --> RETURN[返回给用户代码]
    RETURN --> END
    
    style NETWORK_ERROR fill:#ffcccc
    style HTTP_ERROR fill:#ffcccc
    style BIZ_ERROR fill:#ffcccc
    style SUCCESS fill:#ccffcc
    style RETURN fill:#ccffcc
```

---

## 5. 配置优先级层次图

```mermaid
graph TD
    subgraph Priority["配置优先级（从低到高）"]
        L1[Level 1: SDK 默认配置]
        L2[Level 2: 实例初始化配置]
        L3[Level 3: API 级别配置]
        L4[Level 4: 单次请求配置]
        
        L1 -->|覆盖| L2
        L2 -->|覆盖| L3
        L3 -->|覆盖| L4
    end
    
    subgraph Example["配置示例"]
        DEFAULT["默认:<br/>timeout: 30000"]
        INSTANCE["初始化:<br/>timeout: 10000<br/>baseURL: '/api'"]
        API_LEVEL["API 级:<br/>uploadFile.timeout: 60000"]
        REQUEST["请求级:<br/>{ timeout: 5000 }"]
        
        DEFAULT -.->|合并| INSTANCE
        INSTANCE -.->|合并| API_LEVEL
        API_LEVEL -.->|合并| REQUEST
    end
    
    REQUEST --> FINAL[最终配置:<br/>timeout: 5000<br/>baseURL: '/api']
    
    style L4 fill:#90EE90
    style FINAL fill:#FFD700
```

---

## 6. 插件系统架构图

```mermaid
graph LR
    subgraph SDK["API Client SDK"]
        CORE[核心实例]
        PLUGIN_MANAGER[插件管理器]
        HOOK_REQ[请求钩子链]
        HOOK_RES[响应钩子链]
        HOOK_ERR[错误钩子链]
    end
    
    subgraph Plugins["插件生态"]
        P1[日志插件]
        P2[缓存插件]
        P3[重试插件]
        P4[Mock 插件]
        P5[监控插件]
        P_CUSTOM[自定义插件...]
    end
    
    CORE --> PLUGIN_MANAGER
    
    P1 -->|注册| PLUGIN_MANAGER
    P2 -->|注册| PLUGIN_MANAGER
    P3 -->|注册| PLUGIN_MANAGER
    P4 -->|注册| PLUGIN_MANAGER
    P5 -->|注册| PLUGIN_MANAGER
    P_CUSTOM -->|注册| PLUGIN_MANAGER
    
    PLUGIN_MANAGER --> HOOK_REQ
    PLUGIN_MANAGER --> HOOK_RES
    PLUGIN_MANAGER --> HOOK_ERR
    
    style PLUGIN_MANAGER fill:#FFE4B5
    style P1 fill:#E0FFFF
    style P2 fill:#E0FFFF
    style P3 fill:#E0FFFF
    style P4 fill:#E0FFFF
    style P5 fill:#E0FFFF
```

---

## 7. 多包策略对比

```mermaid
graph TB
    subgraph Strategy_A["策略 A: 单包"]
        SINGLE["@company/api-client"]
        SINGLE --> USER_API_A[UserAPI]
        SINGLE --> ORDER_API_A[OrderAPI]
        SINGLE --> PAYMENT_API_A[PaymentAPI]
        SINGLE --> CORE_A[Core]
    end
    
    subgraph Strategy_B["策略 B: 多包"]
        CORE_B["@company/api-client-core"]
        USER_PKG["@company/api-client-user"]
        ORDER_PKG["@company/api-client-order"]
        PAYMENT_PKG["@company/api-client-payment"]
        
        USER_PKG --> CORE_B
        ORDER_PKG --> CORE_B
        PAYMENT_PKG --> CORE_B
    end
    
    subgraph Comparison["对比"]
        SINGLE_PROS["单包优点:<br/>• 简单直接<br/>• 统一升级<br/>• 版本一致"]
        SINGLE_CONS["单包缺点:<br/>• 体积大<br/>• 全量更新<br/>• 按需加载困难"]
        
        MULTI_PROS["多包优点:<br/>• 按需引入<br/>• 独立升级<br/>• 体积优化"]
        MULTI_CONS["多包缺点:<br/>• 版本管理复杂<br/>• 依赖关系维护"]
    end
    
    style SINGLE fill:#FFE4E1
    style CORE_B fill:#E1FFE4
    style SINGLE_PROS fill:#90EE90
    style MULTI_PROS fill:#90EE90
    style SINGLE_CONS fill:#FFB6C1
    style MULTI_CONS fill:#FFB6C1
```

---

## 8. 版本发布流程图

```mermaid
graph TD
    START[代码变更] --> GIT_COMMIT[Git Commit]
    GIT_COMMIT --> CI_TRIGGER[触发 CI Pipeline]
    
    CI_TRIGGER --> LINT[代码规范检查]
    LINT --> TEST[运行测试]
    TEST --> TYPE_CHECK[TypeScript 类型检查]
    
    TYPE_CHECK --> CHECK_PASS{检查是否通过?}
    CHECK_PASS -->|失败| NOTIFY_FAIL[通知开发者]
    CHECK_PASS -->|通过| DETECT_CHANGE[检测变更类型]
    
    NOTIFY_FAIL --> END_FAIL[流程结束]
    
    DETECT_CHANGE --> VERSION_DECISION{版本号决策}
    VERSION_DECISION -->|Breaking Changes| MAJOR[Major 版本升级<br/>x.0.0]
    VERSION_DECISION -->|新功能| MINOR[Minor 版本升级<br/>0.x.0]
    VERSION_DECISION -->|Bug 修复| PATCH[Patch 版本升级<br/>0.0.x]
    
    MAJOR --> BUILD
    MINOR --> BUILD
    PATCH --> BUILD[构建打包]
    
    BUILD --> GEN_CHANGELOG[生成 Changelog]
    GEN_CHANGELOG --> GIT_TAG[打 Git Tag]
    
    GIT_TAG --> PUBLISH_BETA[发布到 NPM<br/>@beta 标签]
    PUBLISH_BETA --> BETA_TEST[Beta 测试]
    
    BETA_TEST --> BETA_OK{测试是否通过?}
    BETA_OK -->|失败| FIX_BUG[修复问题]
    BETA_OK -->|通过| PUBLISH_LATEST[更新 @latest 标签]
    
    FIX_BUG --> GIT_COMMIT
    
    PUBLISH_LATEST --> UPDATE_DOC[更新文档]
    UPDATE_DOC --> NOTIFY_USERS[通知用户]
    
    NOTIFY_USERS --> END_SUCCESS[发布完成]
    
    style CHECK_PASS fill:#FFE4B5
    style BETA_OK fill:#FFE4B5
    style END_SUCCESS fill:#90EE90
    style END_FAIL fill:#FFB6C1
```

---

## 9. 数据转换流程（Request & Response）

```mermaid
graph LR
    subgraph Request["请求数据转换"]
        R1[用户代码<br/>驼峰命名<br/>{ userName: 'Alice' }]
        R2[业务适配层<br/>下划线命名<br/>{ user_name: 'Alice' }]
        R3[添加业务字段<br/>{ user_name: 'Alice'<br/>tenant_id: 'xxx' }]
        R4[序列化 JSON<br/>'{"user_name":"Alice",...}']
        
        R1 --> R2
        R2 --> R3
        R3 --> R4
        R4 --> SERVER[发送到服务器]
    end
    
    subgraph Response["响应数据转换"]
        SERVER2[接收服务器响应] --> S1
        S1['{"code":0,"data":{...}}']
        S2[解析 JSON<br/>检查业务码]
        S3[提取 data<br/>{ user_name: 'Alice'<br/>created_at: '2025-10-30' }]
        S4[格式转换<br/>{ userName: 'Alice'<br/>createdAt: Date }]
        S5[返回用户代码<br/>类型安全]
        
        S1 --> S2
        S2 --> S3
        S3 --> S4
        S4 --> S5
    end
    
    style R1 fill:#E1F5FF
    style S5 fill:#E1F5FF
    style SERVER fill:#FFE4B5
    style SERVER2 fill:#FFE4B5
```

---

## 10. 团队使用场景图

```mermaid
graph TB
    subgraph Company["公司架构"]
        SDK[公共 API Client SDK]
    end
    
    subgraph TeamA["团队 A - 电商前台"]
        APP_A[商城 Web App]
        ADAPTER_A[团队 A 适配器<br/>• 商品数据格式化<br/>• 购物车逻辑]
        
        SDK --> ADAPTER_A
        ADAPTER_A --> APP_A
    end
    
    subgraph TeamB["团队 B - 后台管理"]
        APP_B[管理后台 App]
        ADAPTER_B[团队 B 适配器<br/>• 权限检查<br/>• 操作日志]
        
        SDK --> ADAPTER_B
        ADAPTER_B --> APP_B
    end
    
    subgraph TeamC["团队 C - 移动端"]
        APP_C[React Native App]
        ADAPTER_C[团队 C 适配器<br/>• 离线缓存<br/>• 弱网优化]
        
        SDK --> ADAPTER_C
        ADAPTER_C --> APP_C
    end
    
    subgraph Backend["后端服务"]
        API[统一 API Gateway]
        SERVICE_USER[用户服务]
        SERVICE_ORDER[订单服务]
        SERVICE_PRODUCT[商品服务]
        
        API --> SERVICE_USER
        API --> SERVICE_ORDER
        API --> SERVICE_PRODUCT
    end
    
    APP_A --> API
    APP_B --> API
    APP_C --> API
    
    style SDK fill:#FFD700
    style ADAPTER_A fill:#FFE4E1
    style ADAPTER_B fill:#E1FFE4
    style ADAPTER_C fill:#E1E4FF
```

---

## 11. 完整生命周期图

```mermaid
stateDiagram-v2
    [*] --> 规范定义: 后端定义 API
    
    规范定义 --> 规范验证: 导出 OpenAPI
    规范验证 --> 代码生成: 验证通过
    规范验证 --> 规范定义: 验证失败
    
    代码生成 --> 本地测试: 生成完成
    本地测试 --> 代码生成: 测试失败
    本地测试 --> CI构建: 测试通过
    
    CI构建 --> 发布Beta: 构建成功
    CI构建 --> 本地测试: 构建失败
    
    发布Beta --> Beta测试: 发布成功
    Beta测试 --> 发布正式版: 测试通过
    Beta测试 --> 修复问题: 发现问题
    修复问题 --> 本地测试: 修复完成
    
    发布正式版 --> 用户安装: 发布到 NPM
    
    用户安装 --> 集成到应用: npm install
    集成到应用 --> 开发使用: 初始化配置
    开发使用 --> 生产运行: 应用上线
    
    生产运行 --> 监控反馈: 持续运行
    监控反馈 --> 需求收集: 发现问题/需求
    需求收集 --> 规范定义: 规划新版本
    
    生产运行 --> [*]: 服务下线
    
    note right of 规范定义
        后端团队职责
    end note
    
    note right of 代码生成
        自动化流程
    end note
    
    note right of 开发使用
        前端团队使用
    end note
    
    note right of 监控反馈
        持续优化
    end note
```

---

## 12. 技术栈选型对比

```mermaid
graph TB
    subgraph CodeGen["代码生成工具"]
        CG1[openapi-typescript<br/>✓ 仅类型<br/>✓ 轻量<br/>✗ 需手写函数]
        CG2[openapi-generator<br/>✓ 完整 SDK<br/>✓ 多语言<br/>✗ 配置复杂]
        CG3[swagger-typescript-api<br/>✓ TS + Axios<br/>✓ 开箱即用<br/>✓ 推荐]
        CG4[orval<br/>✓ React Query 集成<br/>✓ 现代化<br/>✗ 学习成本]
    end
    
    subgraph BuildTool["构建工具"]
        BT1[Rollup<br/>✓ 库打包专用<br/>✓ Tree-shaking 好<br/>✓ 推荐]
        BT2[Webpack<br/>✓ 生态丰富<br/>✗ 配置复杂<br/>✗ 体积大]
        BT3[Vite<br/>✓ 开发体验好<br/>✓ 构建快<br/>✓ 可选]
    end
    
    subgraph TestTool["测试工具"]
        TT1[Vitest<br/>✓ 快速<br/>✓ 兼容 Jest<br/>✓ 推荐]
        TT2[Jest<br/>✓ 成熟稳定<br/>✗ 速度较慢]
        TT3[MSW<br/>✓ Mock 服务器<br/>✓ 推荐用于集成测试]
    end
    
    style CG3 fill:#90EE90
    style BT1 fill:#90EE90
    style TT1 fill:#90EE90
    style TT3 fill:#90EE90
```

---

## 使用说明

### 在 Markdown 查看器中查看

这些图表使用 Mermaid 语法编写，可以在以下平台直接渲染：

1. **GitHub / GitLab**: 直接预览 README.md
2. **VS Code**: 安装 "Markdown Preview Mermaid Support" 插件
3. **Typora**: 内置支持 Mermaid
4. **Notion**: 使用 Mermaid 代码块
5. **在线工具**: https://mermaid.live/

### 导出为图片

访问 https://mermaid.live/ 并粘贴图表代码，可以导出为 PNG/SVG。

---

## 总结

通过这些可视化图表，我们可以清晰地看到：

1. **架构层次**: 6 个清晰的层次，职责分明
2. **数据流向**: 从 OpenAPI 到最终用户的完整流程
3. **错误处理**: 多级错误处理和恢复机制
4. **扩展能力**: 插件系统和适配器的灵活性
5. **版本管理**: 完整的 CI/CD 和发布流程
6. **团队协作**: 多团队共享基础能力，各自扩展

这套工作流设计充分考虑了：
- ✅ 自动化程度
- ✅ 类型安全
- ✅ 可扩展性
- ✅ 可维护性
- ✅ 团队协作
- ✅ 持续优化
