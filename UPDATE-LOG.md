# 更新日志

## 2025-10-30 - 新增流程图示文档

### 🎯 核心变更

新增详细的流程关系与输入输出图示文档：**FLOW-AND-IO-DIAGRAMS.md**

### 📊 文档内容

提供 **10 个详细的可视化图表**，包括：

1. **整体流程关系图** - 展示 8 个阶段的关系
2. **详细输入输出流程图** - 每个阶段的输入、处理、输出
3. **角色交互序列图** - 开发者、工具、系统之间的交互
4. **数据流转详细图** - 从 API 设计到前端使用的完整数据流
5. **文件系统输入输出图** - 输入文件和输出文件的关系
6. **CI/CD 流程详细图** - 自动化构建和发布的详细步骤
7. **包依赖关系图** - NPM 包之间的依赖关系
8. **运行时数据流图** - API 调用的完整过程
9. **错误处理流程图** - 各种错误的处理流程
10. **完整生命周期状态图** - 从创建到运行的状态转换

### 💡 核心价值

- ✅ **一图胜千言** - 快速理解整个系统
- ✅ **清晰的输入输出** - 每个环节的数据流转一目了然
- ✅ **完整的流程** - 从开发到生产的全流程
- ✅ **培训材料** - 可用于团队培训和新人入职

### 📚 文档统计

- **文档大小**: 22KB
- **代码行数**: 742 行
- **图表数量**: 10 个 Mermaid 图表
- **覆盖范围**: 完整的生命周期

---

## 2025-10-30 - 命名方案优化

### 🎯 核心变更

基于用户反馈，优化了 NPM 包的命名方案：

**旧方案**: `@company/user-center-api`  
**新方案**: `@company.api/user-center` ✅

### 📝 变更详情

#### 1. NPM Scope 优化
- **旧**: `@company`
- **新**: `@company.api` ✅

**原因**:
- ✅ Scope 本身就表明这是 API 包
- ✅ 包名可以直接使用服务名，无需 `-api` 后缀
- ✅ 支持扩展为多个 scope（`@company.ui`, `@company.utils` 等）

#### 2. 包名简化
- **旧**: `user-center-api`（包含 `-api` 后缀）
- **新**: `user-center` ✅（纯服务名）

**原因**:
- ✅ 与后端服务名完全一致
- ✅ 更简洁，无冗余
- ✅ scope 已经表明分类，无需后缀

### 📦 完整映射表

| 旧命名 | 新命名 | 说明 |
|--------|--------|------|
| `@company/user-center-api` | `@company.api/user-center` | 用户中心 |
| `@company/order-api` | `@company.api/order` | 订单服务 |
| `@company/payment-api` | `@company.api/payment` | 支付服务 |
| `@company/api-core` | `@company.api/core` | 核心层 |

### 📚 新增文档

1. **NAMING-CONVENTION.md** - 详细的命名规范说明
2. **NAMING-EVOLUTION.md** - 命名方案的演进历史
3. **UPDATE-LOG.md** - 本更新日志

### 🔄 更新的文档

所有文档已全面更新，采用新的命名方案：

1. ✅ README.md
2. ✅ SCAFFOLD-AND-INDEPENDENCE-DESIGN.md
3. ✅ DESIGN-SUMMARY.md
4. ✅ NAMING-CONVENTION.md (新增)
5. ✅ NAMING-EVOLUTION.md (新增)

### 💡 使用示例

#### 创建新项目
```bash
npm create @company/api
# 输入服务名: user-center
# 生成 NPM 包: @company.api/user-center ✅
```

#### 安装依赖
```bash
npm install @company.api/user-center
npm install @company.api/order
npm install @company.api/core
```

#### 代码中使用
```typescript
import { createHttpClient } from '@company.api/core'
import { UserCenterApi } from '@company.api/user-center'
import { OrderApi } from '@company.api/order'

const httpClient = createHttpClient({ /* ... */ })
const userCenterApi = new UserCenterApi(httpClient)
const orderApi = new OrderApi(httpClient)
```

### 🎉 优势总结

相比旧方案，新方案的优势：

1. **更简洁** ✅
   - 从 `@company/user-center-api` (27 字符)
   - 到 `@company.api/user-center` (26 字符)
   - 且无任何冗余信息

2. **Scope 语义化** ✅
   - `@company.api` 专门用于 API 包
   - 可扩展为 `@company.ui`, `@company.utils` 等

3. **包名即服务名** ✅
   - 包名 `user-center` 与后端服务名完全一致
   - 无需记忆额外的后缀或前缀

4. **符合最佳实践** ✅
   - 参考 `@babel/core`, `@angular/core`, `@nestjs/core` 等
   - 大型项目的标准做法

### 🔗 相关文档

- [NAMING-CONVENTION.md](./NAMING-CONVENTION.md) - 命名规范详细说明
- [NAMING-EVOLUTION.md](./NAMING-EVOLUTION.md) - 命名方案演进历史
- [SCAFFOLD-AND-INDEPENDENCE-DESIGN.md](./SCAFFOLD-AND-INDEPENDENCE-DESIGN.md) - 脚手架设计

---

## 之前的更新

### 2025-10-30 - 初始设计

创建了完整的 API 客户端工作流设计方案：

1. API-CLIENT-WORKFLOW-DESIGN.md - 主设计文档
2. INTERACTION-DETAILS.md - 交互细节说明
3. VISUAL-WORKFLOW.md - 可视化流程图
4. README.md - 项目总览
5. SCAFFOLD-AND-INDEPENDENCE-DESIGN.md - 脚手架与独立性设计
6. DESIGN-SUMMARY.md - 设计总结

---

**维护者**: API Client Team  
**最后更新**: 2025-10-30
