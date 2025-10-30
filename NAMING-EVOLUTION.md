# 命名方案演进历史

## 演进过程

### 版本 1.0（初始版本）❌
```
项目名: api-client-user-center
NPM包: @company/api-client-user-center
```
**问题**：
- ❌ 名称过长（31 个字符）
- ❌ `api-client` 前缀冗余
- ❌ 不够直观

### 版本 2.0（第一次优化）⚠️
```
项目名: user-center-api
NPM包: @company/user-center-api
```
**改进**：
- ✅ 服务名在前，更直观
- ✅ 缩短到 27 个字符
- ✅ 与后端服务名更接近

**残留问题**：
- ⚠️ 包名仍包含 `-api` 后缀
- ⚠️ 所有包都在 `@company` scope 下，缺乏分类

### 版本 3.0（最终版本）✅✅✅
```
项目名: user-center-api
NPM包: @company.api/user-center
Scope: @company.api
```
**完美方案**：
- ✅✅✅ **Scope 语义化**：`@company.api` 专门用于 API 包
- ✅✅ **包名即服务名**：`user-center` 与后端服务名完全一致
- ✅✅ **极致简洁**：26 个字符，无任何冗余
- ✅ **命名空间清晰**：支持扩展到其他 scope（`@company.ui`, `@company.utils`）
- ✅ **便于分类管理**：在 NPM 中自动按 scope 分组

---

## 完整对比表

| 方面 | v1.0 初始版本 | v2.0 第一次优化 | v3.0 最终版本 ✅ |
|------|--------------|----------------|-----------------|
| **项目目录** | `api-client-user-center` | `user-center-api` | `user-center-api` |
| **NPM 包名** | `@company/api-client-user-center` | `@company/user-center-api` | `@company.api/user-center` |
| **NPM Scope** | `@company` | `@company` | `@company.api` |
| **字符数** | 31 字符 | 27 字符 | 26 字符 |
| **包名后缀** | `-api-client-user-center` | `-user-center-api` | 无后缀，纯服务名 |
| **语义清晰度** | ⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **扩展性** | ⭐ | ⭐⭐ | ⭐⭐⭐⭐⭐ |

---

## 实际使用对比

### v1.0 初始版本
```bash
# 安装
npm install @company/api-client-user-center
npm install @company/api-client-order
npm install @company/api-client-payment

# 导入
import { UserCenterApi } from '@company/api-client-user-center'
import { OrderApi } from '@company/api-client-order'
import { PaymentApi } from '@company/api-client-payment'
```

### v2.0 第一次优化
```bash
# 安装
npm install @company/user-center-api
npm install @company/order-api
npm install @company/payment-api

# 导入
import { UserCenterApi } from '@company/user-center-api'
import { OrderApi } from '@company/order-api'
import { PaymentApi } from '@company/payment-api'
```

### v3.0 最终版本 ✅
```bash
# 安装
npm install @company.api/user-center
npm install @company.api/order
npm install @company.api/payment
npm install @company.api/core

# 导入
import { createHttpClient } from '@company.api/core'
import { UserCenterApi } from '@company.api/user-center'
import { OrderApi } from '@company.api/order'
import { PaymentApi } from '@company.api/payment'
```

**对比结果**：
- 包名更短，更易读
- Scope 本身就表明了分类
- 与后端服务名完全一致

---

## 命名空间组织

### v3.0 支持的多 Scope 架构

```
@company.api/          ← API 相关的包
├── core
├── user-center
├── order
├── payment
└── product

@company.ui/           ← UI 组件库
├── button
├── modal
├── table
└── form

@company.utils/        ← 工具函数库
├── date
├── string
├── number
└── validation

@company.hooks/        ← React Hooks 库
├── use-user
├── use-auth
└── use-permission

@company.config/       ← 配置包
├── eslint
├── prettier
└── typescript
```

**优势**：
1. **清晰分类**：一眼就能看出包的类型
2. **避免冲突**：不同类型的包可以使用相同的名称
3. **便于管理**：在 NPM 中自动分组
4. **符合惯例**：与主流框架的做法一致（如 `@babel`, `@angular`, `@nestjs`）

---

## 迁移建议

### 对于新项目
直接使用 v3.0 最终版本：
```bash
npm create @company/api
# 输入服务名: user-center
# 自动生成: @company.api/user-center
```

### 对于现有项目
如果使用了 v1.0 或 v2.0，建议迁移：

#### 步骤 1: 创建新包
```bash
npm create @company/api
# 使用相同的服务名
```

#### 步骤 2: 迁移代码和规范
```bash
# 复制 OpenAPI 规范
cp old-project/openapi/*.yaml new-project/openapi/

# 复制自定义代码（如适配器）
cp -r old-project/src/adapters new-project/src/
```

#### 步骤 3: 发布新包
```bash
git tag v1.0.0
git push origin v1.0.0
```

#### 步骤 4: 更新依赖项目
```typescript
// 旧方式
import { Api } from '@company/user-center-api'

// 新方式
import { Api } from '@company.api/user-center'
```

#### 步骤 5: 弃用旧包
```bash
npm deprecate @company/user-center-api "已迁移到 @company.api/user-center"
```

---

## 为什么选择这个方案？

### 1. 学习业界最佳实践

许多成功的大型项目都采用类似的多 Scope 策略：

```
Babel:
  @babel/core
  @babel/preset-env
  @babel/plugin-transform-runtime

Angular:
  @angular/core
  @angular/common
  @angular/forms

NestJS:
  @nestjs/core
  @nestjs/common
  @nestjs/platform-express

TypeScript ESLint:
  @typescript-eslint/parser
  @typescript-eslint/eslint-plugin
```

### 2. NPM 官方推荐

NPM 官方文档推荐使用 Scope 来组织包：
- 避免命名冲突
- 更清晰的包分类
- 更好的访问控制（对于私有包）

### 3. 实际需求驱动

我们的需求：
- ✅ 多个服务的 API 包（user, order, payment...）
- ✅ 公共核心层（core）
- ✅ 可能扩展到其他类型的包（ui, utils...）

使用 `@company.api` scope 是最自然的选择。

---

## FAQ

### Q: 为什么不用 `@company/api-user-center`？
**A**: 虽然也能工作，但有以下问题：
- `api-` 前缀是冗余的（scope 已经表明是 API 包）
- 不如 `@company.api/user-center` 简洁
- scope 无法表达分类信息

### Q: 是否必须使用 `@company.api`？
**A**: 不是必须的，但强烈推荐：
- ✅ 如果你的组织有多种类型的 NPM 包，使用多 scope 是最佳实践
- ⚠️ 如果只有 API 包，使用 `@company` 也可以接受
- ❌ 但无论如何，都不要在包名中加 `-api` 后缀（冗余）

### Q: 如何在私有 NPM Registry 中设置 scope？
**A**: 大多数私有 Registry（Verdaccio, Nexus, Artifactory）都支持 scope，无需预先创建：
```bash
# 配置 Registry
npm config set @company.api:registry https://npm.company.com

# 发布时自动创建 scope
npm publish
```

### Q: 包名中的点号会有问题吗？
**A**: 不会。NPM 完全支持在 scope 中使用点号：
- `@company.api/xxx` ✅ 合法
- `@company-api/xxx` ✅ 合法（但不如点号直观）
- `@company_api/xxx` ✅ 合法（但下划线不常用）

---

## 总结

| 特性 | v1.0 | v2.0 | v3.0 ✅ |
|------|------|------|---------|
| 简洁性 | ❌ | ⚠️ | ✅✅✅ |
| 语义化 | ❌ | ⚠️ | ✅✅✅ |
| 可扩展性 | ❌ | ⚠️ | ✅✅✅ |
| 与后端一致 | ❌ | ⚠️ | ✅✅✅ |
| 符合最佳实践 | ❌ | ⚠️ | ✅✅✅ |

**最终推荐**：`@company.api/user-center` ✅✅✅

---

**文档版本**: 1.0  
**最后更新**: 2025-10-30  
**作者**: API Client Team
