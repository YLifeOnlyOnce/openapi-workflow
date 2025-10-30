# 流程关系与输入输出图示

本文档提供详细的图示，说明各个流程的关系、输入输出和数据流向。

---

## 1. 整体流程关系图

```mermaid
graph TB
    subgraph "阶段 1: 项目创建"
        A1[开发者使用脚手架]
        A2[输入: 服务名称]
        A3[输出: 标准化项目]
    end
    
    subgraph "阶段 2: OpenAPI 维护"
        B1[团队维护 YAML 文件]
        B2[输入: API 设计]
        B3[输出: OpenAPI 规范]
    end
    
    subgraph "阶段 3: 代码生成"
        C1[执行代码生成脚本]
        C2[输入: OpenAPI YAML]
        C3[输出: TS 类型和 API 函数]
    end
    
    subgraph "阶段 4: 本地开发"
        D1[开发和测试]
        D2[输入: 生成的代码]
        D3[输出: 完整的 API 包]
    end
    
    subgraph "阶段 5: 版本发布"
        E1[打 Git Tag]
        E2[输入: 项目代码 + Tag]
        E3[输出: 触发 CI/CD]
    end
    
    subgraph "阶段 6: CI/CD 构建"
        F1[自动构建和测试]
        F2[输入: 代码 + Tag]
        F3[输出: NPM 包]
    end
    
    subgraph "阶段 7: NPM 发布"
        G1[发布到 NPM Registry]
        G2[输入: 构建产物]
        G3[输出: 可安装的包]
    end
    
    subgraph "阶段 8: 前端集成"
        H1[前端应用安装使用]
        H2[输入: NPM 包]
        H3[输出: 类型安全的 API 调用]
    end
    
    A1 --> A2 --> A3
    A3 --> B1
    B1 --> B2 --> B3
    B3 --> C1
    C1 --> C2 --> C3
    C3 --> D1
    D1 --> D2 --> D3
    D3 --> E1
    E1 --> E2 --> E3
    E3 --> F1
    F1 --> F2 --> F3
    F3 --> G1
    G1 --> G2 --> G3
    G3 --> H1
    H1 --> H2 --> H3
    
    style A1 fill:#e1f5ff
    style B1 fill:#fff4e1
    style C1 fill:#ffe1f5
    style D1 fill:#f5ffe1
    style E1 fill:#ffe1e1
    style F1 fill:#e1e1ff
    style G1 fill:#ffe1ff
    style H1 fill:#e1ffe1
```

---

## 2. 详细输入输出流程图

```mermaid
flowchart TD
    Start([开始]) --> Input1

    subgraph Stage1 ["🔨 阶段 1: 项目创建"]
        Input1["📥 输入:<br/>• 服务名称 (user-center)<br/>• 配置选项"]
        Process1["⚙️ 处理:<br/>• 脚手架生成项目结构<br/>• 创建配置文件<br/>• 初始化 Git 仓库"]
        Output1["📤 输出:<br/>• 项目目录: user-center-api/<br/>• package.json<br/>• CI/CD 配置<br/>• 空的 openapi/ 目录"]
        Input1 --> Process1 --> Output1
    end

    Output1 --> Input2

    subgraph Stage2 ["📝 阶段 2: OpenAPI 编写"]
        Input2["📥 输入:<br/>• API 设计需求<br/>• 后端接口文档"]
        Process2["⚙️ 处理:<br/>• 编写 OpenAPI 规范<br/>• 定义 paths, schemas<br/>• 添加文档说明"]
        Output2["📤 输出:<br/>• openapi/user-center.yaml<br/>• 完整的 API 定义"]
        Input2 --> Process2 --> Output2
    end

    Output2 --> Input3

    subgraph Stage3 ["🔄 阶段 3: 代码生成"]
        Input3["📥 输入:<br/>• OpenAPI YAML 文件<br/>• 代码生成配置"]
        Process3["⚙️ 处理:<br/>• swagger-typescript-api<br/>• 解析 OpenAPI 规范<br/>• 生成 TS 代码"]
        Output3["📤 输出:<br/>• src/generated/api/<br/>• src/generated/models/<br/>• src/generated/types/<br/>• TypeScript 类型定义"]
        Input3 --> Process3 --> Output3
    end

    Output3 --> Input4

    subgraph Stage4 ["💻 阶段 4: 本地开发"]
        Input4["📥 输入:<br/>• 生成的代码<br/>• 核心层依赖"]
        Process4["⚙️ 处理:<br/>• 编写适配器<br/>• 添加测试<br/>• 本地调试"]
        Output4["📤 输出:<br/>• 完整的 src/ 目录<br/>• 测试用例<br/>• 准备发布的代码"]
        Input4 --> Process4 --> Output4
    end

    Output4 --> Input5

    subgraph Stage5 ["🏷️ 阶段 5: 打标签"]
        Input5["📥 输入:<br/>• 完成的代码<br/>• 版本号"]
        Process5["⚙️ 处理:<br/>• git tag v1.2.0<br/>• git push origin v1.2.0"]
        Output5["📤 输出:<br/>• Git Tag<br/>• 触发 CI/CD"]
        Input5 --> Process5 --> Output5
    end

    Output5 --> Input6

    subgraph Stage6 ["🤖 阶段 6: CI/CD"]
        Input6["📥 输入:<br/>• Git 仓库<br/>• Git Tag<br/>• CI/CD 配置"]
        Process6["⚙️ 处理:<br/>• 校验 OpenAPI<br/>• 生成代码<br/>• 运行测试<br/>• 构建 NPM 包"]
        Output6["📤 输出:<br/>• dist/ 构建产物<br/>• .tgz 包文件<br/>• Changelog"]
        Input6 --> Process6 --> Output6
    end

    Output6 --> Input7

    subgraph Stage7 ["📦 阶段 7: NPM 发布"]
        Input7["📥 输入:<br/>• 构建产物<br/>• NPM Token<br/>• package.json"]
        Process7["⚙️ 处理:<br/>• npm publish<br/>• 发布到 Registry<br/>• 创建 Release"]
        Output7["📤 输出:<br/>• @company.api/user-center@1.2.0<br/>• NPM 页面<br/>• GitHub Release"]
        Input7 --> Process7 --> Output7
    end

    Output7 --> Input8

    subgraph Stage8 ["🎯 阶段 8: 前端使用"]
        Input8["📥 输入:<br/>• NPM 包<br/>• 前端项目"]
        Process8["⚙️ 处理:<br/>• npm install<br/>• 导入 API<br/>• 调用接口"]
        Output8["📤 输出:<br/>• 类型安全的 API 调用<br/>• 业务功能实现"]
        Input8 --> Process8 --> Output8
    end

    Output8 --> End([结束])

    classDef inputStyle fill:#e3f2fd,stroke:#1976d2,stroke-width:2px
    classDef processStyle fill:#fff3e0,stroke:#f57c00,stroke-width:2px
    classDef outputStyle fill:#e8f5e9,stroke:#388e3c,stroke-width:2px
    
    class Input1,Input2,Input3,Input4,Input5,Input6,Input7,Input8 inputStyle
    class Process1,Process2,Process3,Process4,Process5,Process6,Process7,Process8 processStyle
    class Output1,Output2,Output3,Output4,Output5,Output6,Output7,Output8 outputStyle
```

---

## 3. 角色交互序列图

```mermaid
sequenceDiagram
    autonumber
    
    actor Dev as 开发者
    participant Scaffold as 脚手架工具
    participant Project as 项目仓库
    participant YAML as OpenAPI 规范
    participant Generator as 代码生成器
    participant Git as Git 仓库
    participant CI as CI/CD 系统
    participant NPM as NPM Registry
    participant FE as 前端应用
    
    Note over Dev,FE: 阶段 1: 项目创建
    Dev->>Scaffold: npm create @company/api
    Scaffold->>Dev: 询问配置信息
    Dev->>Scaffold: 输入服务名: user-center
    Scaffold->>Project: 生成项目结构
    Scaffold->>Dev: ✅ 项目创建成功
    
    Note over Dev,FE: 阶段 2: OpenAPI 编写
    Dev->>YAML: 编写 openapi/user-center.yaml
    YAML-->>Dev: 📄 完整的 API 定义
    
    Note over Dev,FE: 阶段 3: 代码生成
    Dev->>Generator: npm run generate
    Generator->>YAML: 读取 OpenAPI 规范
    YAML-->>Generator: 返回规范内容
    Generator->>Project: 生成 TS 代码
    Generator-->>Dev: ✅ 代码生成完成
    
    Note over Dev,FE: 阶段 4: 本地开发
    Dev->>Project: 编写适配器和测试
    Dev->>Project: npm run build
    Project-->>Dev: ✅ 构建成功
    
    Note over Dev,FE: 阶段 5: 版本发布
    Dev->>Git: git tag v1.2.0
    Dev->>Git: git push origin v1.2.0
    Git-->>CI: 🚀 触发 CI/CD (Tag 事件)
    
    Note over Dev,FE: 阶段 6: CI/CD 构建
    CI->>Git: 拉取代码
    CI->>YAML: 校验 OpenAPI 规范
    YAML-->>CI: ✅ 规范有效
    CI->>Generator: 生成代码
    CI->>CI: 运行测试
    CI->>CI: 构建 NPM 包
    CI-->>Dev: 📧 构建成功通知
    
    Note over Dev,FE: 阶段 7: NPM 发布
    CI->>NPM: npm publish
    NPM-->>CI: ✅ 发布成功
    CI->>Git: 创建 GitHub Release
    CI-->>Dev: 📧 发布成功通知
    
    Note over Dev,FE: 阶段 8: 前端使用
    FE->>NPM: npm install @company.api/user-center
    NPM-->>FE: 下载包
    FE->>FE: import { UserCenterApi }
    FE->>FE: 调用 API
    FE-->>FE: ✅ 功能实现
```

---

## 4. 数据流转详细图

```mermaid
graph LR
    subgraph "源头"
        A1[后端 API 设计]
    end
    
    subgraph "OpenAPI 规范"
        B1[openapi/user-center.yaml]
        B2["内容:<br/>- paths<br/>- schemas<br/>- parameters<br/>- responses"]
    end
    
    subgraph "代码生成"
        C1[swagger-typescript-api]
        C2["生成:<br/>- TypeScript 接口<br/>- API 函数<br/>- 数据模型"]
    end
    
    subgraph "核心层"
        D1[@company.api/core]
        D2["提供:<br/>- HttpClient<br/>- 拦截器<br/>- 错误处理"]
    end
    
    subgraph "业务适配"
        E1[src/adapters/]
        E2["定制:<br/>- 数据转换<br/>- 业务逻辑<br/>- 缓存策略"]
    end
    
    subgraph "构建产物"
        F1[dist/]
        F2["包含:<br/>- index.esm.js<br/>- index.cjs.js<br/>- index.d.ts"]
    end
    
    subgraph "NPM 包"
        G1[@company.api/user-center]
        G2["版本: 1.2.0<br/>大小: ~50KB"]
    end
    
    subgraph "前端应用"
        H1[业务代码]
        H2["使用:<br/>- 导入 API<br/>- 调用方法<br/>- 处理响应"]
    end
    
    A1 -->|设计| B1
    B1 -->|规范定义| B2
    B2 -->|输入| C1
    C1 -->|生成代码| C2
    C2 -->|依赖| D1
    D1 -->|提供能力| D2
    D2 -->|集成| E1
    E1 -->|添加逻辑| E2
    E2 -->|构建| F1
    F1 -->|打包| F2
    F2 -->|发布| G1
    G1 -->|版本信息| G2
    G2 -->|安装| H1
    H1 -->|集成使用| H2
    
    style A1 fill:#ffebee
    style B1 fill:#e3f2fd
    style C1 fill:#f3e5f5
    style D1 fill:#e8f5e9
    style E1 fill:#fff3e0
    style F1 fill:#fce4ec
    style G1 fill:#e0f2f1
    style H1 fill:#f1f8e9
```

---

## 5. 文件系统输入输出图

```mermaid
graph TD
    subgraph "项目根目录: user-center-api/"
        Root[项目根目录]
    end
    
    subgraph "输入文件"
        I1[openapi/user-center.yaml]
        I2[src/core/http-client.ts]
        I3[src/adapters/user-adapter.ts]
        I4[package.json]
        I5[tsconfig.json]
        I6[rollup.config.js]
        
        I1 -.->|定义 API| Gen
        I2 -.->|提供核心功能| Gen
        I3 -.->|业务逻辑| Gen
        I4 -.->|依赖配置| Gen
        I5 -.->|TS 配置| Gen
        I6 -.->|构建配置| Gen
    end
    
    subgraph "处理过程"
        Gen["代码生成 + 构建<br/>━━━━━━━━━━<br/>npm run generate<br/>npm run build"]
    end
    
    subgraph "生成文件 (临时)"
        T1[src/generated/api/]
        T2[src/generated/models/]
        T3[src/generated/types/]
        
        Gen -->|生成| T1
        Gen -->|生成| T2
        Gen -->|生成| T3
    end
    
    subgraph "输出文件 (最终产物)"
        O1[dist/index.esm.js]
        O2[dist/index.cjs.js]
        O3[dist/index.d.ts]
        O4[dist/package.json]
        O5[CHANGELOG.md]
        
        Gen -->|构建| O1
        Gen -->|构建| O2
        Gen -->|构建| O3
        Gen -->|构建| O4
        Gen -->|生成| O5
    end
    
    subgraph "发布到 NPM"
        NPM["@company.api/user-center@1.2.0"]
        
        O1 -.->|打包| NPM
        O2 -.->|打包| NPM
        O3 -.->|打包| NPM
        O4 -.->|打包| NPM
    end
    
    Root --> I1
    Root --> I2
    Root --> I3
    Root --> I4
    Root --> I5
    Root --> I6
    
    style I1 fill:#e3f2fd
    style I2 fill:#e3f2fd
    style I3 fill:#e3f2fd
    style I4 fill:#e3f2fd
    style I5 fill:#e3f2fd
    style I6 fill:#e3f2fd
    
    style T1 fill:#fff3e0
    style T2 fill:#fff3e0
    style T3 fill:#fff3e0
    
    style O1 fill:#e8f5e9
    style O2 fill:#e8f5e9
    style O3 fill:#e8f5e9
    style O4 fill:#e8f5e9
    style O5 fill:#e8f5e9
    
    style Gen fill:#f3e5f5
    style NPM fill:#fce4ec
```

---

## 6. CI/CD 流程详细图

```mermaid
graph TB
    Start([Git Push Tag]) --> Trigger
    
    subgraph "触发阶段"
        Trigger["📥 输入:<br/>• Git Tag: v1.2.0<br/>• 代码提交"]
        Trigger --> Checkout
    end
    
    subgraph "准备阶段"
        Checkout["⚙️ 检出代码<br/>• git clone<br/>• 切换到 tag"]
        NodeSetup["⚙️ 设置环境<br/>• Node.js 20<br/>• npm ci"]
        Checkout --> NodeSetup
    end
    
    subgraph "验证阶段"
        Validate["⚙️ 验证规范<br/>• npm run validate<br/>• 检查 OpenAPI"]
        Generate["⚙️ 生成代码<br/>• npm run generate<br/>• 创建 TS 文件"]
        NodeSetup --> Validate
        Validate --> Generate
    end
    
    subgraph "测试阶段"
        Lint["⚙️ 代码检查<br/>• ESLint<br/>• Prettier"]
        TypeCheck["⚙️ 类型检查<br/>• tsc --noEmit"]
        UnitTest["⚙️ 单元测试<br/>• Vitest"]
        Generate --> Lint
        Lint --> TypeCheck
        TypeCheck --> UnitTest
    end
    
    subgraph "构建阶段"
        Build["⚙️ 构建包<br/>• npm run build<br/>• Rollup 打包"]
        UnitTest --> Build
    end
    
    subgraph "发布阶段"
        Version["⚙️ 更新版本<br/>• 提取 tag 版本号<br/>• 更新 package.json"]
        Changelog["⚙️ 生成日志<br/>• CHANGELOG.md"]
        Publish["⚙️ 发布 NPM<br/>• npm publish"]
        Release["⚙️ 创建 Release<br/>• GitHub Release"]
        Build --> Version
        Version --> Changelog
        Changelog --> Publish
        Publish --> Release
    end
    
    subgraph "通知阶段"
        Notify["📤 输出:<br/>• 发送邮件/Slack<br/>• 通知相关人员"]
        Release --> Notify
    end
    
    Notify --> End([完成])
    
    subgraph "失败处理"
        Fail["❌ 失败处理<br/>• 记录错误<br/>• 发送通知<br/>• 停止流程"]
    end
    
    Validate -.->|失败| Fail
    Lint -.->|失败| Fail
    TypeCheck -.->|失败| Fail
    UnitTest -.->|失败| Fail
    Build -.->|失败| Fail
    Publish -.->|失败| Fail
    
    style Trigger fill:#e3f2fd
    style Validate fill:#fff3e0
    style UnitTest fill:#fff3e0
    style Build fill:#f3e5f5
    style Publish fill:#e8f5e9
    style Notify fill:#e8f5e9
    style Fail fill:#ffebee
```

---

## 7. 包依赖关系图

```mermaid
graph TD
    subgraph "NPM Registry"
        Core[@company.api/core<br/>━━━━━━━━━━<br/>版本: 2.0.0<br/>提供: 核心功能]
    end
    
    subgraph "API 包们"
        UserCenter[@company.api/user-center<br/>━━━━━━━━━━<br/>版本: 1.2.0<br/>依赖: core ^2.0.0]
        Order[@company.api/order<br/>━━━━━━━━━━<br/>版本: 2.1.0<br/>依赖: core ^2.0.0]
        Payment[@company.api/payment<br/>━━━━━━━━━━<br/>版本: 1.0.5<br/>依赖: core ^2.0.0]
    end
    
    subgraph "前端应用"
        WebApp[Web 应用<br/>━━━━━━━━━━<br/>安装:<br/>- @company.api/core<br/>- @company.api/user-center<br/>- @company.api/order]
        MobileApp[移动端应用<br/>━━━━━━━━━━<br/>安装:<br/>- @company.api/core<br/>- @company.api/payment]
    end
    
    Core -->|提供基础能力| UserCenter
    Core -->|提供基础能力| Order
    Core -->|提供基础能力| Payment
    
    UserCenter -->|使用| WebApp
    Order -->|使用| WebApp
    Core -->|使用| WebApp
    
    Payment -->|使用| MobileApp
    Core -->|使用| MobileApp
    
    style Core fill:#e8f5e9
    style UserCenter fill:#e3f2fd
    style Order fill:#e3f2fd
    style Payment fill:#e3f2fd
    style WebApp fill:#fff3e0
    style MobileApp fill:#fff3e0
```

---

## 8. 运行时数据流图

```mermaid
sequenceDiagram
    autonumber
    
    participant User as 用户
    participant UI as 前端 UI
    participant API as @company.api/user-center
    participant Adapter as 业务适配层
    participant Core as @company.api/core
    participant Axios as Axios
    participant Backend as 后端服务
    
    Note over User,Backend: 用户发起请求
    User->>UI: 点击"获取用户信息"
    
    Note over User,Backend: 前端调用 API
    UI->>API: userCenterApi.getProfile({ userId: '123' })
    
    Note over User,Backend: 业务适配层处理
    API->>Adapter: transformRequest(data)
    Adapter->>Adapter: • 格式转换<br/>• 添加业务字段
    Adapter-->>API: 转换后的数据
    
    Note over User,Backend: 核心层发送请求
    API->>Core: httpClient.get('/users/123', data)
    Core->>Core: 请求拦截器<br/>• 添加 Token<br/>• 添加 Headers
    
    Core->>Axios: axios.get(...)
    Axios->>Backend: HTTP GET /api/users/123
    
    Note over User,Backend: 后端处理并返回
    Backend-->>Axios: HTTP 200 { code: 0, data: {...} }
    
    Note over User,Backend: 核心层处理响应
    Axios-->>Core: response
    Core->>Core: 响应拦截器<br/>• 检查状态码<br/>• 解析 JSON<br/>• 数据解包
    
    Note over User,Backend: 业务适配层转换
    Core-->>Adapter: rawData
    Adapter->>Adapter: transformResponse(rawData)<br/>• 格式转换<br/>• 类型转换
    Adapter-->>API: 转换后的数据
    
    Note over User,Backend: 返回给前端
    API-->>UI: { id: '123', name: 'Alice', ... }
    UI->>UI: 更新界面
    UI-->>User: 显示用户信息
    
    rect rgb(200, 255, 200)
        Note over User,Backend: ✅ 请求成功完成
    end
```

---

## 9. 错误处理流程图

```mermaid
graph TD
    Start([API 请求]) --> Request
    
    Request[发送请求] --> Check{响应状态}
    
    Check -->|网络错误| NetworkError
    Check -->|HTTP 4xx| ClientError
    Check -->|HTTP 5xx| ServerError
    Check -->|HTTP 200| CheckBiz{检查业务码}
    
    subgraph "网络错误处理"
        NetworkError[NetworkError<br/>━━━━━━━━━━<br/>原因: 无网络/超时]
        NetworkError --> Retry1{是否重试?}
        Retry1 -->|是| Wait1[等待后重试]
        Retry1 -->|否| Log1[记录日志]
        Wait1 --> Request
        Log1 --> Notify1[通知用户]
    end
    
    subgraph "客户端错误处理"
        ClientError[ClientError<br/>━━━━━━━━━━<br/>4xx 错误]
        ClientError --> Check401{401?}
        Check401 -->|是| Login[跳转登录]
        Check401 -->|否| Check403{403?}
        Check403 -->|是| Permission[显示权限提示]
        Check403 -->|否| Other4xx[其他 4xx 错误]
        Other4xx --> Log2[记录日志]
        Log2 --> Notify2[提示用户]
    end
    
    subgraph "服务器错误处理"
        ServerError[ServerError<br/>━━━━━━━━━━<br/>5xx 错误]
        ServerError --> Retry2{是否重试?}
        Retry2 -->|是| Wait2[等待后重试]
        Retry2 -->|否| Log3[记录日志]
        Wait2 --> Request
        Log3 --> Notify3[通知用户]
    end
    
    subgraph "业务错误处理"
        CheckBiz -->|code !== 0| BizError[BusinessError<br/>━━━━━━━━━━<br/>业务错误码]
        CheckBiz -->|code === 0| Success[提取数据]
        BizError --> MapError[错误码映射]
        MapError --> Log4[记录日志]
        Log4 --> Notify4[提示用户]
    end
    
    Success --> Transform[数据转换]
    Transform --> Return[返回给调用方]
    Return --> End([完成])
    
    Notify1 --> End
    Login --> End
    Permission --> End
    Notify2 --> End
    Notify3 --> End
    Notify4 --> End
    
    style NetworkError fill:#ffebee
    style ClientError fill:#fff3e0
    style ServerError fill:#ffebee
    style BizError fill:#fff3e0
    style Success fill:#e8f5e9
    style Return fill:#e8f5e9
```

---

## 10. 完整生命周期状态图

```mermaid
stateDiagram-v2
    [*] --> 项目创建: 使用脚手架
    
    项目创建 --> OpenAPI编写: 项目结构生成
    
    OpenAPI编写 --> 代码生成: YAML 完成
    OpenAPI编写 --> OpenAPI编写: 修改规范
    
    代码生成 --> 本地开发: TS 代码生成
    
    本地开发 --> 本地测试: 编写适配器
    本地开发 --> 本地开发: 继续开发
    
    本地测试 --> 本地开发: 测试失败
    本地测试 --> 准备发布: 测试通过
    
    准备发布 --> 打Tag: 提交代码
    
    打Tag --> CI触发: push tag
    
    CI触发 --> CI验证: 拉取代码
    
    CI验证 --> CI测试: 验证通过
    CI验证 --> 修复问题: 验证失败
    
    CI测试 --> CI构建: 测试通过
    CI测试 --> 修复问题: 测试失败
    
    CI构建 --> NPM发布: 构建成功
    CI构建 --> 修复问题: 构建失败
    
    修复问题 --> 本地开发: 修复并提交
    
    NPM发布 --> 前端安装: 发布成功
    
    前端安装 --> 集成使用: npm install
    
    集成使用 --> 生产运行: 应用上线
    
    生产运行 --> 监控反馈: 持续运行
    
    监控反馈 --> OpenAPI编写: 发现需求/问题
    监控反馈 --> [*]: 服务下线
    
    note right of 项目创建
        输入: 服务名称
        输出: 项目结构
    end note
    
    note right of OpenAPI编写
        输入: API 设计
        输出: YAML 文件
    end note
    
    note right of 代码生成
        输入: YAML
        输出: TS 代码
    end note
    
    note right of NPM发布
        输入: 构建产物
        输出: NPM 包
    end note
```

---

## 总结

本文档提供了 10 个详细的图示，涵盖：

1. ✅ **整体流程关系** - 8 个阶段的关系
2. ✅ **详细输入输出** - 每个阶段的输入、处理、输出
3. ✅ **角色交互序列** - 开发者、工具、系统之间的交互
4. ✅ **数据流转** - 从设计到使用的完整数据流
5. ✅ **文件系统** - 输入文件和输出文件的关系
6. ✅ **CI/CD 流程** - 自动化构建和发布的详细步骤
7. ✅ **包依赖关系** - NPM 包之间的依赖
8. ✅ **运行时数据流** - API 调用的完整过程
9. ✅ **错误处理** - 各种错误的处理流程
10. ✅ **完整生命周期** - 从创建到运行的状态转换

这些图示可以帮助团队成员快速理解整个系统的工作原理和各个环节的关系。

---

**文档版本**: 1.0  
**最后更新**: 2025-10-30  
**作者**: API Client Team
