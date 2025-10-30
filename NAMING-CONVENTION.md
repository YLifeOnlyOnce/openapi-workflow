# 命名规范说明

## 项目命名规范

### 为什么采用这种命名方式？

经过多次优化，我们采用了最简洁、最直观的命名方式：
- **项目目录**: `{服务名}-api` - 明确表明这是 API 项目
- **NPM 包名**: `@company.api/{服务名}` - 使用专门的 scope，包名更简洁

### 命名模式

| 元素 | 命名方式 | 示例 |
|------|---------|------|
| **服务名称** | 小写字母，连字符分隔 | `user-center`, `order`, `payment` |
| **项目目录** | `{服务名}-api` | `user-center-api`, `order-api`, `payment-api` |
| **NPM 包名** | `@company.api/{服务名}` | `@company.api/user-center`, `@company.api/order` |
| **NPM Scope** | `@company.api` | 专门用于 API 包的 scope |
| **Git 仓库** | `{服务名}-api` | `user-center-api.git` |

### 完整示例

#### 示例 1: 用户中心服务

```
输入服务名: user-center

生成结果:
  项目目录: user-center-api/
  NPM 包名: @company.api/user-center
  NPM Scope: @company.api
  仓库地址: https://github.com/company/user-center-api.git
  OpenAPI 文件: openapi/user-center.yaml
```

#### 示例 2: 订单服务

```
输入服务名: order

生成结果:
  项目目录: order-api/
  NPM 包名: @company.api/order
  NPM Scope: @company.api
  仓库地址: https://github.com/company/order-api.git
  OpenAPI 文件: openapi/order.yaml
```

#### 示例 3: 支付服务

```
输入服务名: payment

生成结果:
  项目目录: payment-api/
  NPM 包名: @company.api/payment
  NPM Scope: @company.api
  仓库地址: https://github.com/company/payment-api.git
  OpenAPI 文件: openapi/payment.yaml
```

### 命名优势

#### 1. 极致简洁 ✅✅✅
```
❌ 最初方式: @company/api-client-user-center  (31 字符)
❌ 改进方式: @company/user-center-api        (27 字符)
✅ 最佳方式: @company.api/user-center        (26 字符，且无冗余)
```

#### 2. Scope 语义化 ✅✅
```
@company.api/user-center  ← scope 本身就表明这是 API 包
@company.api/order        ← 一眼就知道这是 API 类的包
@company.ui/button        ← 可以有其他 scope 用于不同分类
```

#### 3. 包名与服务名完全一致 ✅✅
```
后端服务: user-center
API 包:   @company.api/user-center  ← 包名就是服务名，完美对应！
```

#### 4. 更好的命名空间组织 ✅✅
```bash
# NPM 包自动按 scope 分组
@company.api/
  ├── user-center
  ├── order
  ├── payment
  └── product

@company.ui/
  ├── button
  ├── modal
  └── table

@company.utils/
  ├── date
  └── string
```

#### 5. 便于管理和查找 ✅
```bash
# 在项目中安装
npm install @company.api/user-center
npm install @company.api/order

# 清晰的分类，易于管理
```

### 核心层命名

核心层提供公共的 HTTP 客户端、拦截器等基础功能：

```
项目目录: api-core/
NPM 包名: @company.api/core
NPM Scope: @company.api
```

使用相同的 scope，保持一致性。`core` 表示这是 API 的核心库。

### 脚手架工具命名

```
NPM 包名: @company/create-api
使用命令: npm create @company/api
```

脚手架工具使用独立的 scope `@company`，因为它不是 API 包本身。

### 前端应用中的使用

#### 安装
```bash
# 极致简洁的包名
npm install @company.api/user-center
npm install @company.api/order
npm install @company.api/payment
npm install @company.api/core  # 核心层
```

#### 导入
```typescript
// 清晰的导入语句
import { createHttpClient } from '@company.api/core'
import { UserCenterApi } from '@company.api/user-center'
import { OrderApi } from '@company.api/order'
import { PaymentApi } from '@company.api/payment'

// 初始化
const httpClient = createHttpClient({ /* ... */ })

// API 实例命名也很直观
const userCenterApi = new UserCenterApi(httpClient)
const orderApi = new OrderApi(httpClient)
const paymentApi = new PaymentApi(httpClient)
```

#### package.json
```json
{
  "dependencies": {
    "@company.api/core": "^2.0.0",
    "@company.api/user-center": "^1.2.0",
    "@company.api/order": "^2.1.0",
    "@company.api/payment": "^1.0.5"
  }
}
```

### 命名规则总结

#### ✅ 最佳实践
- **Scope 语义化**: `@company.api/xxx` ✅✅✅
- **包名即服务名**: `user-center` 而不是 `user-center-api` ✅✅
- **命名空间清晰**: 不同类型的包使用不同的 scope ✅
- **与后端一致**: 包名与后端服务名完全相同 ✅

#### ❌ 不推荐的命名
- **冗余前缀**: `api-client-user-center` ❌
- **包名带后缀**: `@company/user-center-api` ❌ (虽然已经不错，但还能更好)
- **Scope 不明确**: 所有包都在 `@company` 下 ❌

### 特殊情况处理

#### 多单词服务名
使用连字符连接，保持小写：
```
服务名: user-management
项目: user-management-api
NPM: @company.api/user-management
```

#### 缩写
如果服务名是缩写，保持小写：
```
服务名: crm
项目: crm-api
NPM: @company.api/crm
```

#### 版本号
**强烈推荐**使用 NPM 的版本管理，而不是在包名中体现版本：
```
✅ 推荐: @company.api/user-center@2.0.0
❌ 不推荐: @company.api/user-center-v2
```

#### NPM Scope 设置

在私有 NPM Registry 或 npmjs.com 上创建 `@company.api` scope：

```bash
# 如果使用 npmjs.com
# 1. 登录 NPM
npm login

# 2. 创建组织 (如果还没有)
# 访问 https://www.npmjs.com/org/create
# 创建组织名: company.api

# 3. 发布包时会自动使用该 scope
npm publish --access public
```

如果使用私有 Registry (如 Verdaccio, Nexus)，通常不需要预先创建 scope。

### 脚手架交互设计

脚手架会自动根据服务名生成所有相关名称：

```bash
$ npm create @company/api

◆  服务名称 (Service name):
│  user-center
│  (将生成项目: user-center-api, NPM包: @company.api/user-center)
```

用户只需要输入服务名称，其他名称自动生成，确保命名的一致性。

### 迁移指南

#### 从旧命名方式迁移

| 旧命名 | 新命名 | 说明 |
|--------|--------|------|
| `@company/api-client-user` | `@company.api/user-center` | 最初版本 → 最新版本 |
| `@company/user-center-api` | `@company.api/user-center` | 中间版本 → 最新版本 |

#### 迁移步骤

##### 1. 创建新项目
```bash
npm create @company/api
# 输入服务名: user-center
```

##### 2. 迁移 OpenAPI 规范
```bash
cp old-project/openapi/user-api.yaml user-center-api/openapi/user-center.yaml
```

##### 3. 更新 package.json
```json
{
  "name": "@company.api/user-center",
  "version": "1.0.0"
}
```

##### 4. 发布新包
```bash
cd user-center-api
git tag v1.0.0
git push origin v1.0.0

# CI/CD 会自动发布到 @company.api/user-center
```

##### 5. 更新前端应用
```typescript
// 旧方式 1
import { userApi } from '@company/api-client-user'

// 旧方式 2
import { UserCenterApi } from '@company/user-center-api'

// 新方式 ✅
import { UserCenterApi } from '@company.api/user-center'
import { createHttpClient } from '@company.api/core'

const httpClient = createHttpClient({ /* ... */ })
const userCenterApi = new UserCenterApi(httpClient)
```

##### 6. 弃用旧包
```bash
# 标记旧包为 deprecated
npm deprecate @company/api-client-user "已迁移到 @company.api/user-center"
npm deprecate @company/user-center-api "已迁移到 @company.api/user-center"
```

### 为什么 @company.api 比 @company 更好？

#### 命名空间隔离
```
@company.api/xxx     ← API 相关的包
@company.ui/xxx      ← UI 组件库
@company.utils/xxx   ← 工具函数库
@company.hooks/xxx   ← React Hooks 库
@company.config/xxx  ← 配置包
```

#### 更清晰的包分类
在 NPM 或私有 Registry 中，包会按 scope 自动分组，一目了然。

#### 避免命名冲突
不同类型的包使用不同的 scope，避免命名冲突：
```
@company.api/user     ← API 包
@company.ui/user      ← 用户相关的 UI 组件
```

#### 符合大型项目最佳实践
许多大型项目都采用这种多 scope 的方式组织包：
```
@babel/core
@babel/preset-env

@angular/core
@angular/common

@nestjs/core
@nestjs/common
```

---

**文档版本**: 1.0  
**最后更新**: 2025-10-30
