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

## 🚀 快速开始

### 第一步：理解架构
1. 先阅读 [API-CLIENT-WORKFLOW-DESIGN.md](./API-CLIENT-WORKFLOW-DESIGN.md) 的「整体架构概览」和「工作流环节拆解」部分
2. 查看 [VISUAL-WORKFLOW.md](./VISUAL-WORKFLOW.md) 中的「整体架构图」，形成直观认知

### 第二步：深入细节
1. 阅读 [API-CLIENT-WORKFLOW-DESIGN.md](./API-CLIENT-WORKFLOW-DESIGN.md) 的「关键设计决策」部分
2. 根据你关注的点，查阅 [INTERACTION-DETAILS.md](./INTERACTION-DETAILS.md) 对应的章节

### 第三步：规划实施
1. 参考 [API-CLIENT-WORKFLOW-DESIGN.md](./API-CLIENT-WORKFLOW-DESIGN.md) 的「落地实施路线图」
2. 结合团队实际情况，调整技术选型和实施计划

---

## 🏗️ 架构概览

```
┌─────────────────────────────────────────────────────────────────┐
│                    API Client 工作流全景                          │
└─────────────────────────────────────────────────────────────────┘

后端服务 (OpenAPI)
    ↓
① OpenAPI 规范管理层 (规范采集、验证、版本管理)
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

### 1. 完整的自动化流程
- 从 OpenAPI 规范到可用的 NPM 包，全流程自动化
- CI/CD 集成，规范变更自动触发 SDK 更新

### 2. 端到端的类型安全
- OpenAPI Schema → TypeScript Types
- 完整的类型推导链，减少运行时错误

### 3. 灵活的扩展机制
- **插件系统**: 日志、缓存、重试、监控等可插拔
- **适配器模式**: 各团队可自定义数据转换和业务逻辑

### 4. 分层清晰，职责单一
- 每一层关注点不同，易于维护和测试
- 核心层稳定，扩展层灵活

### 5. 完善的质量保障
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

### 安装
```bash
npm install @company/api-client
```

### 初始化
```typescript
import { createApiClient } from '@company/api-client'

const apiClient = createApiClient({
  baseURL: 'https://api.example.com',
  getToken: () => localStorage.getItem('token'),
  onError: (error) => {
    // 全局错误处理
  }
})
```

### 调用 API
```typescript
// 类型安全的 API 调用
const user = await apiClient.user.getProfile({ userId: '123' })
console.log(user.name) // TypeScript 自动提示
```

---

## 🤔 FAQ

### Q1: 后端没有提供 OpenAPI 规范怎么办？
**A**: 可以根据接口文档手动编写 OpenAPI 规范，或者使用工具从现有代码生成（如 `swagger-jsdoc`）。

### Q2: 单包还是多包策略？
**A**: 初期建议单包策略，简单快速。当服务数量增多（5+ 个）或团队规模扩大后，再考虑拆分为多包。

### Q3: 如何处理 Breaking Changes？
**A**: 
1. 使用工具检测 Breaking Changes
2. 升级 Major 版本号
3. 提供详细的迁移指南
4. 保留旧版本一段时间，给团队迁移缓冲期

### Q4: 团队有特殊需求怎么办？
**A**: 通过业务适配层扩展，不影响核心层。每个团队可以实现自己的 Adapter，注入自定义逻辑。

---

## 📝 关键设计原则

1. **自动化优先**: 能自动化的绝不手动
2. **类型安全**: 充分利用 TypeScript 的类型系统
3. **职责分离**: 分层清晰，每层关注点不同
4. **向后兼容**: API 变更遵循语义化版本
5. **可观测性**: 完善的日志和监控

---

## 🎓 适用场景

### ✅ 适合使用该方案的场景
- 微服务架构，API 数量多
- 多个前端团队共享 API
- 后端提供 OpenAPI 规范
- 需要类型安全的 API 调用
- 追求自动化和标准化

### ⚠️ 不太适合的场景
- API 非常少（< 10 个）
- 后端完全不支持 OpenAPI
- 项目生命周期很短（< 3 个月）
- 团队规模很小（1-2 人）

---

## 🔗 相关资源

- [OpenAPI Specification](https://swagger.io/specification/)
- [Axios Documentation](https://axios-http.com/)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)
- [swagger-typescript-api](https://github.com/acacode/swagger-typescript-api)
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
