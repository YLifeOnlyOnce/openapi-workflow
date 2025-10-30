# 命名规范说明

## 项目命名规范

### 为什么采用简洁的命名方式？

原设计中使用 `api-client-{service}` 的命名方式，但在实际使用中发现这样的命名过于冗长且不够直观。新的命名规范采用 `{service}-api` 的形式，更加简洁明了。

### 命名模式

| 元素 | 命名方式 | 示例 |
|------|---------|------|
| **服务名称** | 小写字母，连字符分隔 | `user-center`, `order`, `payment` |
| **项目目录** | `{服务名}-api` | `user-center-api`, `order-api`, `payment-api` |
| **NPM 包名** | `@company/{服务名}-api` | `@company/user-center-api` |
| **Git 仓库** | `{服务名}-api` | `user-center-api.git` |

### 完整示例

#### 示例 1: 用户中心服务

```
输入服务名: user-center

生成结果:
  项目目录: user-center-api/
  NPM 包名: @company/user-center-api
  仓库地址: https://github.com/company/user-center-api.git
  OpenAPI 文件: openapi/user-center.yaml
```

#### 示例 2: 订单服务

```
输入服务名: order

生成结果:
  项目目录: order-api/
  NPM 包名: @company/order-api
  仓库地址: https://github.com/company/order-api.git
  OpenAPI 文件: openapi/order.yaml
```

#### 示例 3: 支付服务

```
输入服务名: payment

生成结果:
  项目目录: payment-api/
  NPM 包名: @company/payment-api
  仓库地址: https://github.com/company/payment-api.git
  OpenAPI 文件: openapi/payment.yaml
```

### 命名优势

#### 1. 简洁直观 ✅
```
❌ 旧方式: @company/api-client-user-center
✅ 新方式: @company/user-center-api

更短，更易读！
```

#### 2. 语义清晰 ✅
```
@company/user-center-api  ← 一眼就知道这是用户中心的 API
@company/order-api        ← 一眼就知道这是订单的 API
```

#### 3. 符合惯例 ✅
```
后端服务: user-center-service
API 文档: user-center-api-docs
API 包:   user-center-api  ← 保持命名一致性
```

#### 4. 便于管理 ✅
```bash
# NPM 包列表中，按字母排序后很容易找到
@company/order-api
@company/payment-api
@company/product-api
@company/user-center-api

# 而不是
@company/api-client-order
@company/api-client-payment
@company/api-client-product
@company/api-client-user-center
```

### 核心层命名

核心层提供公共的 HTTP 客户端、拦截器等基础功能：

```
项目目录: api-core/
NPM 包名: @company/api-core
```

简洁明了，表示这是 API 相关的核心库。

### 脚手架工具命名

```
NPM 包名: @company/create-api
使用命令: npm create @company/api
```

简短易记，符合 `create-*` 的命名惯例。

### 前端应用中的使用

#### 安装
```bash
# 简洁的包名，易于记忆和输入
npm install @company/user-center-api
npm install @company/order-api
npm install @company/payment-api
```

#### 导入
```typescript
// 清晰的导入语句
import { UserCenterApi } from '@company/user-center-api'
import { OrderApi } from '@company/order-api'
import { PaymentApi } from '@company/payment-api'

// API 实例命名也很直观
const userCenterApi = new UserCenterApi(httpClient)
const orderApi = new OrderApi(httpClient)
const paymentApi = new PaymentApi(httpClient)
```

### 命名规则总结

#### ✅ 推荐的命名
- **服务名在前**: `user-center-api` ✅
- **简洁直观**: `order-api` ✅
- **保持一致**: 与后端服务名称对应 ✅

#### ❌ 不推荐的命名
- **冗余前缀**: `api-client-user-center` ❌
- **过于通用**: `api-client` ❌
- **不一致**: 与后端服务名称不对应 ❌

### 特殊情况处理

#### 多单词服务名
使用连字符连接，保持小写：
```
user-management  → user-management-api
product-catalog  → product-catalog-api
```

#### 缩写
如果服务名是缩写，保持大小写一致性：
```
crm  → crm-api
oa   → oa-api
```

#### 版本号
如果需要在包名中体现版本，放在最后：
```
user-center-api-v2
@company/user-center-api-v2
```

但更推荐使用 NPM 的版本管理，而不是在包名中体现版本。

### 脚手架交互设计

脚手架会自动根据服务名生成所有相关名称：

```bash
$ npm create @company/api

◆  服务名称 (Service name):
│  user-center
│  (将生成项目: user-center-api, NPM包: @company/user-center-api)
```

用户只需要输入服务名称，其他名称自动生成，确保命名的一致性。

### 迁移指南

如果已经使用旧的命名方式，可以按照以下步骤迁移：

#### 1. 创建新项目
```bash
npm create @company/api
# 输入服务名: user-center
```

#### 2. 迁移 OpenAPI 规范
```bash
cp old-project/openapi/user-api.yaml user-center-api/openapi/user-center.yaml
```

#### 3. 发布新包
```bash
cd user-center-api
git tag v1.0.0
git push origin v1.0.0
```

#### 4. 更新前端应用
```typescript
// 旧方式
import { userApi } from '@company/api-client-user'

// 新方式
import { userCenterApi } from '@company/user-center-api'
```

#### 5. 弃用旧包
在 npm 上标记旧包为 deprecated：
```bash
npm deprecate @company/api-client-user "已迁移到 @company/user-center-api"
```

---

**文档版本**: 1.0  
**最后更新**: 2025-10-30
