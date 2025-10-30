# API Client 工作流设计文档

## 📚 文档概览

本项目提供了一套完整的、可供多团队使用的前端 API Client 工作流设计方案。该方案基于 OpenAPI 规范自动生成，使用 Axios 作为 HTTP 客户端,提供类型安全的 API 调用能力。

## 🎯 核心目标

- **标准化**: 统一 API 调用方式和错误处理
- **自动化**: 从 OpenAPI 规范自动生成 TypeScript SDK
- **类型安全**: 端到端的 TypeScript 类型保障
- **可复用**: 多团队共享基础能力
- **可扩展**: 支持插件和适配器机制

## 📖 文档结构

### 1. [API-CLIENT-WORKFLOW-DESIGN.md](./API-CLIENT-WORKFLOW-DESIGN.md)
**主设计文档 - 必读** 📍

详细阐述整个工作流的设计思路，包括：
- ✅ 整体架构概览
- ✅ 6 个核心环节的详细拆解
- ✅ 每个环节的职责、输入、输出
- ✅ 关键设计决策和最佳实践
- ✅ 技术选型建议
- ✅ 质量保障策略
- ✅ 实施路线图

**适合人群**: 架构师、技术负责人、初次了解该方案的开发者

---

### 2. [INTERACTION-DETAILS.md](./INTERACTION-DETAILS.md)
**交互细节文档 - 深入理解**

深入讲解各环节之间的交互关系，包括：
- ✅ 环节间的详细交互方式
- ✅ 请求和响应的数据流转
- ✅ 配置流转和优先级
- ✅ 错误处理流程
- ✅ 扩展机制（插件、适配器）
- ✅ 版本升级和反馈循环

**适合人群**: 需要深入理解实现细节的开发者、负责具体实施的工程师

---

### 3. [VISUAL-WORKFLOW.md](./VISUAL-WORKFLOW.md)
**可视化流程图 - 直观展示**

使用 Mermaid 图表展示工作流，包括：
- ✅ 整体架构图
- ✅ 开发时序图（SDK 生成）
- ✅ 运行时序图（API 调用）
- ✅ 错误处理流程图
- ✅ 配置优先级层次图
- ✅ 插件系统架构图
- ✅ 版本发布流程图
- ✅ 数据转换流程
- ✅ 完整生命周期图

**适合人群**: 所有人，特别是喜欢通过图表理解架构的读者

**查看方式**: 在 GitHub/GitLab 上直接预览，或使用支持 Mermaid 的 Markdown 编辑器

---

### 4. [SCAFFOLD-AND-INDEPENDENCE-DESIGN.md](./SCAFFOLD-AND-INDEPENDENCE-DESIGN.md)
**脚手架与项目独立性 - 实施方案** 🎯

详细说明如何实现项目独立性，包括：
- ✅ 项目独立性架构设计（集中式 vs 独立式）
- ✅ 脚手架工具设计与实现
- ✅ 完整的 CI/CD 自动化流程
- ✅ 通过 Git Tag 触发自动发布
- ✅ 公共核心层设计
- ✅ 多项目依赖关系管理
- ✅ 前端应用集成方式
- ✅ 最佳实践和常见问题

**适合人群**: 需要实施独立 API 客户端项目的团队、DevOps 工程师、架构师

**核心价值**: 
- 各团队独立维护，互不干扰
- 一键创建标准化项目
- 自动化构建和发布流程
- 按需引入，独立升级

---

## 🚀 快速开始

### 第一步：理解整体设计
1. 先阅读 [API-CLIENT-WORKFLOW-DESIGN.md](./API-CLIENT-WORKFLOW-DESIGN.md) 的「整体架构概览」和「工作流环节拆解」部分
2. 查看 [VISUAL-WORKFLOW.md](./VISUAL-WORKFLOW.md) 中的「整体架构图」，形成直观认知

### 第二步：理解项目独立性方案 ⭐
1. **重点阅读** [SCAFFOLD-AND-INDEPENDENCE-DESIGN.md](./SCAFFOLD-AND-INDEPENDENCE-DESIGN.md)
2. 理解集中式管理 vs 独立项目的区别
3. 了解脚手架工具的使用方法
4. 掌握 CI/CD 自动发布流程

### 第三步：深入实现细节
1. 阅读 [INTERACTION-DETAILS.md](./INTERACTION-DETAILS.md) 了解各层交互
2. 根据你关注的点，查阅对应章节的详细说明

### 第四步：开始实施
1. 使用脚手架创建第一个 API 客户端项目
2. 配置 CI/CD 流程
3. 通过 Git Tag 触发第一次发布
4. 在前端应用中集成测试

---

## 🏗️ 架构概览

### 项目独立性架构

```
┌─────────────────────────────────────────────────────────────────┐
│                   独立 API 客户端项目架构                          │
└─────────────────────────────────────────────────────────────────┘

脚手架工具 (@company/create-api-client)
    ↓ 一键创建
    ├─────────────┬─────────────┬─────────────┐
    │             │             │             │
团队 A 项目    团队 B 项目    团队 C 项目    团队 N 项目
api-client-   api-client-   api-client-   api-client-
user          order         payment       ...
    │             │             │             │
    │ 各自维护    │ 各自维护    │ 各自维护    │ 各自维护
    │ OpenAPI     │ OpenAPI     │ OpenAPI     │ OpenAPI
    │ YAML        │ YAML        │ YAML        │ YAML
    ↓             ↓             ↓             ↓
Git Tag       Git Tag       Git Tag       Git Tag
v1.2.0        v2.1.0        v1.0.5        vX.Y.Z
    ↓             ↓             ↓             ↓
CI/CD 自动   CI/CD 自动   CI/CD 自动   CI/CD 自动
构建发布      构建发布      构建发布      构建发布
    ↓             ↓             ↓             ↓
NPM 仓库      NPM 仓库      NPM 仓库      NPM 仓库
@company/     @company/     @company/     @company/
api-client-   api-client-   api-client-   api-client-
user          order         payment       ...
    │             │             │             │
    └─────────────┴─────────────┴─────────────┘
                    ↓
              前端应用（按需引入）
```

### 单个项目内部架构

```
API Client 项目内部
    ↓
① OpenAPI 规范管理 (项目内 YAML 文件)
    ↓
② 代码生成层 (类型生成、API 函数生成)
    ↓
③ 基础请求层 (Axios 封装、拦截器、错误处理)
    ↓
④ 业务适配层 (数据转换、业务逻辑、团队定制)
    ↓
⑤ SDK 封装层 (打包构建、发布到 NPM)
    ↓
⑥ 消费使用层 (React/Vue/Angular 应用)
```

---

## 🔑 核心亮点

### 1. 项目独立性 🎯 **NEW**
- **各团队独立维护**：每个项目组维护自己的 OpenAPI YAML 文件
- **独立版本管理**：各项目独立发版，互不影响
- **按需引入**：前端应用只安装需要的 API 客户端包
- **快速迭代**：无需协调多个团队，快速发布新版本

### 2. 脚手架工具 🛠️ **NEW**
- **一键创建项目**：通过 `npm create @company/api-client` 快速创建标准化项目
- **交互式配置**：友好的命令行交互，自动生成配置文件
- **多模板支持**：支持通用、React、Vue 等多种模板
- **CI/CD 自动配置**：自动生成 GitHub Actions / GitLab CI 配置

### 3. 自动化发布流程 🚀 **NEW**
- **Tag 触发发布**：开发者打 Git Tag 即可触发自动发布
- **完整的 CI/CD**：自动构建、测试、发布到 NPM
- **版本管理**：自动根据 Tag 更新版本号
- **通知机制**：发布成功后自动通知相关人员

### 4. 端到端的类型安全
- OpenAPI Schema → TypeScript Types
- 完整的类型推导链，减少运行时错误

### 5. 灵活的扩展机制
- **插件系统**: 日志、缓存、重试、监控等可插拔
- **适配器模式**: 各团队可自定义数据转换和业务逻辑
- **公共核心层**: 抽取通用代码，各项目共享

### 6. 分层清晰，职责单一
- 每一层关注点不同，易于维护和测试
- 核心层稳定，扩展层灵活

### 7. 完善的质量保障
- 自动化测试（单元、集成、类型）
- 规范校验和 Breaking Changes 检测
- 监控埋点和错误上报

---

## 📊 技术栈建议

### 代码生成
- **推荐**: `swagger-typescript-api` (开箱即用，TS + Axios)
- 备选: `openapi-typescript`, `openapi-generator-cli`, `orval`

### 构建打包
- **推荐**: `Rollup` (专为库打包设计)
- 备选: `Vite`, `Webpack`

### 测试
- **推荐**: `Vitest` (快速，兼容 Jest) + `MSW` (Mock 服务器)
- 备选: `Jest`

### 类型检查
- `TypeScript` (strict 模式)
- `tsd` (类型测试)

---

## 🛠️ 实施阶段

### Phase 1: 基础设施 (1-2 周)
- [ ] 搭建 OpenAPI 规范管理仓库
- [ ] 选型并配置代码生成工具
- [ ] 实现核心请求层

### Phase 2: 自动化流程 (1-2 周)
- [ ] 搭建 CI/CD Pipeline
- [ ] 配置 NPM Registry
- [ ] 实现自动发布

### Phase 3: 功能增强 (2-3 周)
- [ ] 实现业务适配层
- [ ] 开发插件机制
- [ ] 完善错误处理和监控

### Phase 4: 文档和推广 (1 周)
- [ ] 编写完整文档
- [ ] 在试点团队中试用
- [ ] 收集反馈并优化

### Phase 5: 全面推广 (持续)
- [ ] 推广到所有团队
- [ ] 持续维护和迭代

---

## 💡 使用示例

### 创建新的 API 客户端项目

```bash
# 使用脚手架创建项目
npm create @company/api-client

# 按照交互式提示填写信息
# ✓ 项目名称: api-client-user
# ✓ 选择代码生成工具: swagger-typescript-api
# ✓ 选择 CI/CD: GitHub Actions
# ✓ 项目创建成功！

# 进入项目
cd api-client-user

# 安装依赖
npm install

# 开发模式
npm run dev
```

### 发布新版本

```bash
# 修改 OpenAPI 规范
vim openapi/user-api.yaml

# 提交代码
git add .
git commit -m "feat: add user profile endpoint"
git push origin main

# 打 Tag 触发自动发布
git tag v1.1.0
git push origin v1.1.0

# CI/CD 自动完成构建和发布
# 几分钟后，新版本就会发布到 NPM
```

### 在前端应用中使用

```bash
# 安装需要的 API 客户端包
npm install @company/api-client-user
npm install @company/api-client-order
```

```typescript
// 初始化
import { createHttpClient } from '@company/api-client-core'
import { UserApi } from '@company/api-client-user'
import { OrderApi } from '@company/api-client-order'

const httpClient = createHttpClient({
  baseURL: 'https://api.example.com',
  getToken: () => localStorage.getItem('token'),
  onError: (error) => {
    // 全局错误处理
  }
})

export const userApi = new UserApi(httpClient)
export const orderApi = new OrderApi(httpClient)

// 调用 API
const user = await userApi.getProfile({ userId: '123' })
console.log(user.name) // TypeScript 自动提示
```

---

## 🤔 FAQ

### Q1: 为什么要采用独立项目架构而不是单一仓库？
**A**: 
- **解耦合**: 各团队独立维护，互不干扰
- **快速迭代**: 无需协调多个团队，随时发布新版本
- **按需引入**: 前端应用只安装需要的包，减小体积
- **独立升级**: 各项目独立升级，不影响其他项目

### Q2: 如何保证各项目的代码规范统一？
**A**: 
1. 使用脚手架工具创建项目，保证初始结构统一
2. 将公共代码抽取到 `@company/api-client-core` 包
3. 在脚手架中集成 ESLint、Prettier 等工具
4. 提供完善的文档和最佳实践指南

### Q3: 多个项目如何共享公共逻辑？
**A**: 
创建公共核心层包 `@company/api-client-core`，包含：
- HTTP 客户端封装
- 拦截器
- 错误处理器
- 工具函数

各项目通过依赖这个包来复用公共代码。

### Q4: 如何处理 Breaking Changes？
**A**: 
1. 使用工具检测 Breaking Changes
2. 升级 Major 版本号（通过 Git Tag）
3. 在 CHANGELOG 中详细说明变更
4. 通知使用该包的团队进行升级

### Q5: CI/CD 发布失败怎么办？
**A**: 
1. 查看 GitHub Actions 日志，定位失败原因
2. 常见问题：OpenAPI 规范验证失败、测试不通过、NPM Token 过期
3. 修复问题后，删除旧 Tag，重新打 Tag 发布
4. 参考文档中的「常见问题」章节

### Q6: 后端没有提供 OpenAPI 规范怎么办？
**A**: 
1. 根据接口文档手动编写 OpenAPI 规范
2. 使用工具从后端代码生成（如 `swagger-jsdoc`）
3. 推动后端团队提供标准的 OpenAPI 规范

---

## 📝 关键设计原则

1. **自动化优先**: 能自动化的绝不手动
2. **类型安全**: 充分利用 TypeScript 的类型系统
3. **职责分离**: 分层清晰，每层关注点不同
4. **向后兼容**: API 变更遵循语义化版本
5. **可观测性**: 完善的日志和监控

---

## 🎓 适用场景

### ✅ 特别适合独立项目架构的场景
- **多团队协作**: 有多个独立的前端团队
- **微服务架构**: 后端有多个独立的微服务
- **频繁迭代**: 各团队需要独立快速迭代
- **按需引入**: 前端应用不需要全部 API
- **独立维护**: 各团队负责不同的业务域

### ✅ 适合使用该方案的一般场景
- API 数量较多（> 20 个端点）
- 后端提供 OpenAPI 规范（或可以生成）
- 需要类型安全的 API 调用
- 追求自动化和标准化
- 项目需要长期维护

### ⚠️ 不太适合的场景
- API 非常少（< 10 个端点）
- 只有一个小团队维护
- 项目生命周期很短（< 3 个月）
- 后端完全不支持 OpenAPI 且无法生成
- 团队规模很小（1-2 人）且无扩展计划

---

## 🔗 相关资源

### 官方文档
- [OpenAPI Specification](https://swagger.io/specification/)
- [Axios Documentation](https://axios-http.com/)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)
- [NPM Documentation](https://docs.npmjs.com/)

### 代码生成工具
- [swagger-typescript-api](https://github.com/acacode/swagger-typescript-api) (推荐)
- [openapi-typescript](https://github.com/drwpow/openapi-typescript)
- [openapi-generator](https://github.com/OpenAPITools/openapi-generator)
- [orval](https://github.com/anymaniax/orval)

### CI/CD 和工具
- [GitHub Actions](https://docs.github.com/en/actions)
- [GitLab CI](https://docs.gitlab.com/ee/ci/)
- [Semantic Versioning](https://semver.org/)
- [Conventional Commits](https://www.conventionalcommits.org/)

### 可视化工具
- [Mermaid Live Editor](https://mermaid.live/)

---

## 📧 反馈和贡献

如果你有任何问题、建议或想法，欢迎：
- 提交 Issue
- 发起 Pull Request
- 在团队内部论坛讨论

---

## 📄 文档版本

- **版本**: 1.0
- **最后更新**: 2025-10-30
- **作者**: API Client Team

---

## 🎉 开始探索

现在就开始阅读 [API-CLIENT-WORKFLOW-DESIGN.md](./API-CLIENT-WORKFLOW-DESIGN.md)，了解完整的设计方案吧！
