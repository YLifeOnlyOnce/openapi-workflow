# OpenAPI API 客户端脚手架与项目独立性设计

## 一、设计目标

### 核心理念
**每个项目组独立维护自己的 API 客户端包，互不干扰**

- ✅ 各项目组维护自己的 OpenAPI YAML 文件
- ✅ 通过脚手架快速创建标准化的 API 客户端项目
- ✅ 独立的 CI/CD 流程，自动构建和发布
- ✅ 通过 Git Tag 触发版本发布
- ✅ 发布到 NPM 仓库，供前端应用使用

### 架构对比

#### ❌ 旧方案：集中式管理
```
单一仓库
  ├── specs/
  │   ├── user-api.yaml
  │   ├── order-api.yaml
  │   └── payment-api.yaml
  └── 统一生成 → @company/api-client (巨石包)

问题：
- 所有团队依赖同一个包，版本升级影响全局
- 变更需要协调多个团队
- 包体积大，无法按需引入
```

#### ✅ 新方案：项目独立
```
团队 A 仓库: api-client-user
  ├── openapi/user-api.yaml
  ├── .github/workflows/publish.yml
  └── 发布 → @company/api-client-user

团队 B 仓库: api-client-order
  ├── openapi/order-api.yaml
  ├── .github/workflows/publish.yml
  └── 发布 → @company/api-client-order

团队 C 仓库: api-client-payment
  ├── openapi/payment-api.yaml
  ├── .github/workflows/publish.yml
  └── 发布 → @company/api-client-payment

优势：
- 各团队独立维护，互不干扰
- 独立版本，按需升级
- 包体积小，按需引入
- 快速迭代，无需协调
```

---

## 二、脚手架设计

### 2.1 脚手架功能

**CLI 工具名称**: `@company/create-api-client`

#### 核心功能
1. **项目初始化**: 快速创建标准化的 API 客户端项目
2. **模板管理**: 提供预设模板（React、Vue、通用等）
3. **配置生成**: 自动生成 CI/CD 配置文件
4. **依赖安装**: 自动安装必要的依赖包

#### 使用方式
```bash
# 方式 1: 通过 npm/pnpm/yarn
npm create @company/api-client

# 方式 2: 通过 npx
npx @company/create-api-client

# 方式 3: 全局安装后使用
npm install -g @company/create-api-client
create-api-client
```

### 2.2 交互式问答

```bash
$ npm create @company/api-client

┌  Create API Client
│
◆  项目名称 (Package name):
│  api-client-user
│
◆  显示名称 (Display name):
│  User API Client
│
◆  描述 (Description):
│  用户服务的 API 客户端
│
◆  团队/作者 (Author):
│  Team A <team-a@company.com>
│
◆  OpenAPI 规范来源:
│  ● 本地 YAML 文件 (我已经有 OpenAPI 规范)
│  ○ 从 URL 获取 (从后端服务自动拉取)
│
◆  OpenAPI 文件路径:
│  ./specs/user-api.yaml
│
◆  选择代码生成工具:
│  ● swagger-typescript-api (推荐)
│  ○ openapi-typescript
│  ○ openapi-generator-cli
│  ○ orval
│
◆  选择框架集成:
│  ● 通用 (Vanilla TypeScript + Axios)
│  ○ React (with React Query)
│  ○ Vue (with Pinia)
│
◆  NPM 包名称 (Package name):
│  @company/api-client-user
│
◆  NPM Registry:
│  https://registry.npmjs.org (或私有 Registry)
│
◆  Git 仓库地址 (可选):
│  https://github.com/company/api-client-user.git
│
◆  选择 CI/CD 平台:
│  ● GitHub Actions
│  ○ GitLab CI
│  ○ Jenkins
│  ○ 无 (手动发布)
│
└  ✓ 项目创建成功！

接下来:
  cd api-client-user
  npm install
  npm run dev
```

### 2.3 项目结构

脚手架生成的项目结构：

```
api-client-user/
├── openapi/                      # OpenAPI 规范目录
│   └── user-api.yaml             # 主 OpenAPI 文件
│
├── src/                          # 源代码目录
│   ├── core/                     # 核心请求层
│   │   ├── http-client.ts        # Axios 封装
│   │   ├── interceptors.ts       # 拦截器
│   │   └── error-handler.ts      # 错误处理
│   │
│   ├── generated/                # 自动生成的代码（.gitignore）
│   │   ├── api/                  # API 函数
│   │   ├── models/               # 数据模型
│   │   └── types/                # TypeScript 类型
│   │
│   ├── adapters/                 # 业务适配层（可选）
│   │   └── example-adapter.ts
│   │
│   ├── plugins/                  # 插件（可选）
│   │   └── example-plugin.ts
│   │
│   └── index.ts                  # 入口文件
│
├── scripts/                      # 脚本目录
│   ├── generate.ts               # 代码生成脚本
│   ├── validate-spec.ts          # 规范验证脚本
│   └── bump-version.ts           # 版本升级脚本
│
├── tests/                        # 测试目录
│   ├── unit/                     # 单元测试
│   ├── integration/              # 集成测试
│   └── mocks/                    # Mock 数据
│
├── .github/                      # GitHub Actions 配置
│   └── workflows/
│       ├── ci.yml                # CI 流程：测试、构建
│       └── publish.yml           # 发布流程：构建、发布到 NPM
│
├── .gitignore
├── .eslintrc.js
├── .prettierrc
├── tsconfig.json
├── rollup.config.js              # 打包配置
├── package.json
├── README.md                     # 使用文档
└── CHANGELOG.md                  # 变更日志
```

### 2.4 package.json 配置

```json
{
  "name": "@company/api-client-user",
  "version": "1.0.0",
  "description": "用户服务的 API 客户端",
  "author": "Team A <team-a@company.com>",
  "license": "MIT",
  
  "main": "dist/index.cjs.js",
  "module": "dist/index.esm.js",
  "types": "dist/index.d.ts",
  
  "exports": {
    ".": {
      "import": "./dist/index.esm.js",
      "require": "./dist/index.cjs.js",
      "types": "./dist/index.d.ts"
    }
  },
  
  "files": [
    "dist",
    "README.md",
    "CHANGELOG.md"
  ],
  
  "scripts": {
    "dev": "npm run generate && npm run build:watch",
    "generate": "tsx scripts/generate.ts",
    "validate": "tsx scripts/validate-spec.ts",
    "build": "npm run generate && rollup -c",
    "build:watch": "rollup -c -w",
    "test": "vitest",
    "test:ci": "vitest run --coverage",
    "lint": "eslint src --ext .ts",
    "format": "prettier --write \"src/**/*.ts\"",
    "typecheck": "tsc --noEmit",
    "prepublishOnly": "npm run validate && npm run test:ci && npm run build"
  },
  
  "dependencies": {
    "axios": "^1.6.0"
  },
  
  "devDependencies": {
    "@company/api-client-core": "^1.0.0",  // 公共核心层（可选）
    "@rollup/plugin-typescript": "^11.1.5",
    "@types/node": "^20.10.0",
    "eslint": "^8.55.0",
    "prettier": "^3.1.0",
    "rollup": "^4.6.0",
    "swagger-typescript-api": "^13.0.0",
    "tsx": "^4.7.0",
    "typescript": "^5.3.3",
    "vitest": "^1.0.0"
  },
  
  "publishConfig": {
    "access": "public",
    "registry": "https://registry.npmjs.org"
  }
}
```

---

## 三、CI/CD 自动化流程

### 3.1 工作流程概览

```
开发者修改 OpenAPI 规范
  ↓
提交代码到 Git 仓库
  ↓
【可选】手动运行测试和构建
  ↓
打 Git Tag (例如: v1.2.3)
  ↓
推送 Tag 到远程仓库
  ↓
触发 GitHub Actions
  ↓
┌─────────────────────────────────┐
│   CI/CD Pipeline 自动执行        │
│                                 │
│  1. 校验 OpenAPI 规范            │
│  2. 生成代码                     │
│  3. 运行测试                     │
│  4. TypeScript 类型检查          │
│  5. 构建 NPM 包                  │
│  6. 生成 Changelog              │
│  7. 发布到 NPM Registry          │
│  8. 创建 GitHub Release         │
│  9. 发送通知                     │
└─────────────────────────────────┘
  ↓
NPM 包发布成功
  ↓
前端应用可以安装新版本
```

### 3.2 GitHub Actions 配置

#### 文件 1: `.github/workflows/ci.yml` (CI 流程)

```yaml
name: CI

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Validate OpenAPI spec
        run: npm run validate
      
      - name: Generate code
        run: npm run generate
      
      - name: Lint code
        run: npm run lint
      
      - name: Type check
        run: npm run typecheck
      
      - name: Run tests
        run: npm run test:ci
      
      - name: Build package
        run: npm run build
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/coverage-final.json
```

#### 文件 2: `.github/workflows/publish.yml` (发布流程)

```yaml
name: Publish to NPM

on:
  push:
    tags:
      - 'v*.*.*'  # 触发条件：当推送 v1.0.0 格式的 tag 时

jobs:
  publish:
    runs-on: ubuntu-latest
    
    permissions:
      contents: write  # 允许创建 Release
      packages: write  # 允许发布包
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          fetch-depth: 0  # 获取完整历史，用于生成 Changelog
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          registry-url: 'https://registry.npmjs.org'
          cache: 'npm'
      
      - name: Extract version from tag
        id: extract_version
        run: |
          TAG=${GITHUB_REF#refs/tags/v}
          echo "VERSION=$TAG" >> $GITHUB_OUTPUT
          echo "Publishing version: $TAG"
      
      - name: Update package.json version
        run: |
          npm version ${{ steps.extract_version.outputs.VERSION }} --no-git-tag-version
      
      - name: Install dependencies
        run: npm ci
      
      - name: Validate OpenAPI spec
        run: npm run validate
      
      - name: Generate code
        run: npm run generate
      
      - name: Run tests
        run: npm run test:ci
      
      - name: Build package
        run: npm run build
      
      - name: Generate Changelog
        id: changelog
        uses: mikepenz/release-changelog-builder-action@v4
        with:
          configuration: '.github/changelog-config.json'
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Publish to NPM
        run: npm publish
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
      
      - name: Create GitHub Release
        uses: softprops/action-gh-release@v1
        with:
          body: ${{ steps.changelog.outputs.changelog }}
          files: |
            dist/**/*
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Send notification
        if: success()
        uses: 8398a7/action-slack@v3
        with:
          status: custom
          custom_payload: |
            {
              text: "🎉 新版本发布成功！",
              attachments: [{
                color: 'good',
                text: `包名: @company/api-client-user\n版本: v${{ steps.extract_version.outputs.VERSION }}\nNPM: https://www.npmjs.com/package/@company/api-client-user`
              }]
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}
```

### 3.3 版本发布操作步骤

#### 开发者操作流程

```bash
# 1. 修改 OpenAPI 规范
vim openapi/user-api.yaml

# 2. 本地测试（可选但推荐）
npm run validate    # 验证规范
npm run generate    # 生成代码
npm run test        # 运行测试
npm run build       # 构建包

# 3. 提交代码
git add .
git commit -m "feat: add new user profile endpoint"
git push origin main

# 4. 打 Tag 发布新版本
# 根据变更类型选择版本号：
#   - 破坏性变更 (Breaking Changes): Major 版本 (v2.0.0)
#   - 新功能 (Features): Minor 版本 (v1.1.0)
#   - Bug 修复 (Fixes): Patch 版本 (v1.0.1)

git tag v1.1.0
git push origin v1.1.0

# 5. 等待 CI/CD 完成（大约 2-5 分钟）
# 访问 GitHub Actions 查看进度: https://github.com/company/api-client-user/actions

# 6. 发布成功后，通知使用者升级
```

#### 自动化版本号管理（推荐）

使用 `standard-version` 或 `release-it` 工具自动管理版本号：

```bash
# 安装工具
npm install --save-dev standard-version

# 在 package.json 中添加脚本
{
  "scripts": {
    "release": "standard-version",
    "release:minor": "standard-version --release-as minor",
    "release:major": "standard-version --release-as major",
    "release:patch": "standard-version --release-as patch"
  }
}

# 使用
npm run release         # 自动判断版本号类型
npm run release:minor   # 强制 minor 版本升级

# 推送 tag
git push --follow-tags origin main
```

---

## 四、公共核心层（可选）

### 4.1 为什么需要公共核心层？

虽然各项目独立，但有些代码是通用的：
- HTTP 客户端封装 (Axios)
- 请求/响应拦截器
- 错误处理器
- 工具函数

**方案**：创建一个公共包 `@company/api-client-core`，供各项目依赖。

### 4.2 公共核心层结构

```
api-client-core/
├── src/
│   ├── http-client.ts         # Axios 封装
│   ├── interceptors/
│   │   ├── auth.ts            # 认证拦截器
│   │   ├── logging.ts         # 日志拦截器
│   │   └── error.ts           # 错误拦截器
│   ├── error-handler.ts       # 错误处理器
│   ├── types.ts               # 公共类型定义
│   └── utils.ts               # 工具函数
├── package.json
└── README.md
```

### 4.3 各项目如何使用核心层

```typescript
// api-client-user/src/index.ts
import { createHttpClient } from '@company/api-client-core'
import { UserApi } from './generated/api'

// 创建 HTTP 客户端
const httpClient = createHttpClient({
  baseURL: 'https://api.example.com',
  timeout: 10000
})

// 注入到生成的 API 中
const userApi = new UserApi(httpClient)

export { userApi }
export * from './generated/models'
export * from './generated/types'
```

### 4.4 核心层独立发布

核心层也是一个独立的 NPM 包：
- 仓库: `api-client-core`
- NPM 包名: `@company/api-client-core`
- 版本管理: 独立版本号
- 发布流程: 与各项目相同的 CI/CD

---

## 五、多项目依赖关系

### 5.1 依赖图

```
@company/api-client-core (核心层)
         ↑
         │ 依赖
         ├──────────┬──────────┬──────────┐
         │          │          │          │
@company/  @company/  @company/  @company/
api-client-  api-client-  api-client-  api-client-
user         order        payment      product
         │          │          │          │
         │          │          │          │
         └──────────┴──────────┴──────────┘
                    ↓
              前端应用 (按需引入)
           import { userApi } from '@company/api-client-user'
           import { orderApi } from '@company/api-client-order'
```

### 5.2 版本兼容性策略

#### 核心层版本策略
```json
{
  "name": "@company/api-client-core",
  "version": "2.0.0"
}
```

#### 各项目依赖核心层
```json
{
  "name": "@company/api-client-user",
  "dependencies": {
    "@company/api-client-core": "^2.0.0"  // 兼容 2.x.x
  }
}
```

**规则**：
- 核心层 Major 版本升级时，各项目需要同步升级
- 核心层 Minor/Patch 版本升级时，各项目无需立即升级（向后兼容）

---

## 六、前端应用集成

### 6.1 安装依赖

```bash
# 只安装需要的 API 客户端包
npm install @company/api-client-user
npm install @company/api-client-order

# 核心层会作为 peer dependency 自动安装
```

### 6.2 使用示例

```typescript
// src/api/index.ts
import { createHttpClient } from '@company/api-client-core'
import { UserApi } from '@company/api-client-user'
import { OrderApi } from '@company/api-client-order'

// 创建全局 HTTP 客户端
const httpClient = createHttpClient({
  baseURL: import.meta.env.VITE_API_BASE_URL,
  timeout: 10000,
  getToken: () => localStorage.getItem('token'),
  onError: (error) => {
    // 全局错误处理
    if (error.status === 401) {
      router.push('/login')
    }
  }
})

// 初始化各 API
export const userApi = new UserApi(httpClient)
export const orderApi = new OrderApi(httpClient)

// 在组件中使用
import { userApi } from '@/api'

async function fetchUserProfile(userId: string) {
  try {
    const user = await userApi.getProfile({ userId })
    console.log(user.name)  // TypeScript 类型提示
  } catch (error) {
    console.error('Failed to fetch user:', error)
  }
}
```

### 6.3 按需加载优化

```typescript
// 使用动态导入，减少初始加载体积
const loadUserApi = async () => {
  const { UserApi } = await import('@company/api-client-user')
  return new UserApi(httpClient)
}

// 在需要时才加载
const userApi = await loadUserApi()
```

---

## 七、脚手架实现细节

### 7.1 脚手架项目结构

```
create-api-client/
├── src/
│   ├── cli.ts                 # CLI 入口
│   ├── prompts.ts             # 交互式问答
│   ├── generator.ts           # 项目生成器
│   ├── templates/             # 模板目录
│   │   ├── base/              # 基础模板
│   │   ├── react/             # React 模板
│   │   └── vue/               # Vue 模板
│   └── utils/
│       ├── file.ts            # 文件操作
│       ├── git.ts             # Git 操作
│       └── npm.ts             # NPM 操作
├── package.json
└── README.md
```

### 7.2 核心代码示例

```typescript
// src/cli.ts
import { Command } from 'commander'
import inquirer from 'inquirer'
import { generateProject } from './generator'

const program = new Command()

program
  .name('create-api-client')
  .description('创建 OpenAPI API 客户端项目')
  .version('1.0.0')
  .action(async () => {
    const answers = await inquirer.prompt([
      {
        type: 'input',
        name: 'projectName',
        message: '项目名称:',
        default: 'api-client-service',
        validate: (input) => {
          if (!/^[a-z0-9-]+$/.test(input)) {
            return '项目名称只能包含小写字母、数字和连字符'
          }
          return true
        }
      },
      {
        type: 'input',
        name: 'displayName',
        message: '显示名称:',
        default: 'API Client'
      },
      {
        type: 'input',
        name: 'description',
        message: '项目描述:',
        default: 'API 客户端'
      },
      {
        type: 'list',
        name: 'generator',
        message: '选择代码生成工具:',
        choices: [
          { name: 'swagger-typescript-api (推荐)', value: 'swagger-typescript-api' },
          { name: 'openapi-typescript', value: 'openapi-typescript' },
          { name: 'openapi-generator-cli', value: 'openapi-generator-cli' },
          { name: 'orval', value: 'orval' }
        ],
        default: 'swagger-typescript-api'
      },
      {
        type: 'list',
        name: 'template',
        message: '选择框架集成:',
        choices: [
          { name: '通用 (Vanilla TypeScript)', value: 'base' },
          { name: 'React (with React Query)', value: 'react' },
          { name: 'Vue (with Pinia)', value: 'vue' }
        ],
        default: 'base'
      },
      {
        type: 'input',
        name: 'packageName',
        message: 'NPM 包名称:',
        default: (answers) => `@company/api-client-${answers.projectName}`,
        validate: (input) => {
          if (!/^@[a-z0-9-~][a-z0-9-._~]*\/[a-z0-9-._~]+$/.test(input)) {
            return 'NPM 包名称格式不正确'
          }
          return true
        }
      },
      {
        type: 'list',
        name: 'cicd',
        message: '选择 CI/CD 平台:',
        choices: [
          { name: 'GitHub Actions', value: 'github' },
          { name: 'GitLab CI', value: 'gitlab' },
          { name: 'Jenkins', value: 'jenkins' },
          { name: '无 (手动发布)', value: 'none' }
        ],
        default: 'github'
      }
    ])

    await generateProject(answers)
  })

program.parse()
```

```typescript
// src/generator.ts
import fs from 'fs-extra'
import path from 'path'
import { execSync } from 'child_process'
import chalk from 'chalk'
import ora from 'ora'

export async function generateProject(config: any) {
  const { projectName, packageName, template, cicd } = config
  const projectPath = path.join(process.cwd(), projectName)

  // 检查目录是否已存在
  if (await fs.pathExists(projectPath)) {
    console.error(chalk.red(`错误: 目录 ${projectName} 已存在`))
    process.exit(1)
  }

  const spinner = ora('正在创建项目...').start()

  try {
    // 1. 创建项目目录
    await fs.ensureDir(projectPath)

    // 2. 复制模板文件
    const templatePath = path.join(__dirname, '../templates', template)
    await fs.copy(templatePath, projectPath)

    // 3. 生成 package.json
    const packageJson = {
      name: packageName,
      version: '1.0.0',
      description: config.description,
      author: config.author || '',
      license: 'MIT',
      main: 'dist/index.cjs.js',
      module: 'dist/index.esm.js',
      types: 'dist/index.d.ts',
      files: ['dist', 'README.md'],
      scripts: {
        dev: 'npm run generate && npm run build:watch',
        generate: 'tsx scripts/generate.ts',
        validate: 'tsx scripts/validate-spec.ts',
        build: 'npm run generate && rollup -c',
        'build:watch': 'rollup -c -w',
        test: 'vitest',
        'test:ci': 'vitest run --coverage',
        lint: 'eslint src --ext .ts',
        format: 'prettier --write "src/**/*.ts"',
        typecheck: 'tsc --noEmit',
        prepublishOnly: 'npm run validate && npm run test:ci && npm run build'
      },
      dependencies: {
        axios: '^1.6.0'
      },
      devDependencies: getDevDependencies(config.generator)
    }

    await fs.writeJSON(
      path.join(projectPath, 'package.json'),
      packageJson,
      { spaces: 2 }
    )

    // 4. 生成 CI/CD 配置
    if (cicd !== 'none') {
      const cicdTemplatePath = path.join(__dirname, '../templates/cicd', cicd)
      await fs.copy(cicdTemplatePath, path.join(projectPath, '.github'))
    }

    // 5. 创建 README.md
    const readme = generateReadme(config)
    await fs.writeFile(path.join(projectPath, 'README.md'), readme)

    // 6. 初始化 Git
    execSync('git init', { cwd: projectPath, stdio: 'ignore' })
    execSync('git add .', { cwd: projectPath, stdio: 'ignore' })
    execSync('git commit -m "chore: initial commit"', { cwd: projectPath, stdio: 'ignore' })

    spinner.succeed('项目创建成功！')

    // 7. 显示后续步骤
    console.log()
    console.log(chalk.green('✓ 项目已创建'))
    console.log()
    console.log('接下来:')
    console.log(chalk.cyan(`  cd ${projectName}`))
    console.log(chalk.cyan('  npm install'))
    console.log(chalk.cyan('  npm run dev'))
    console.log()
    console.log('发布新版本:')
    console.log(chalk.cyan('  git tag v1.0.0'))
    console.log(chalk.cyan('  git push origin v1.0.0'))
    console.log()

  } catch (error) {
    spinner.fail('项目创建失败')
    console.error(chalk.red(error))
    process.exit(1)
  }
}

function getDevDependencies(generator: string) {
  const base = {
    '@rollup/plugin-typescript': '^11.1.5',
    '@types/node': '^20.10.0',
    'eslint': '^8.55.0',
    'prettier': '^3.1.0',
    'rollup': '^4.6.0',
    'tsx': '^4.7.0',
    'typescript': '^5.3.3',
    'vitest': '^1.0.0'
  }

  const generatorDeps = {
    'swagger-typescript-api': { 'swagger-typescript-api': '^13.0.0' },
    'openapi-typescript': { 'openapi-typescript': '^6.7.0' },
    'openapi-generator-cli': { '@openapitools/openapi-generator-cli': '^2.7.0' },
    'orval': { 'orval': '^6.23.0' }
  }

  return {
    ...base,
    ...generatorDeps[generator]
  }
}

function generateReadme(config: any) {
  return `# ${config.displayName}

${config.description}

## 安装

\`\`\`bash
npm install ${config.packageName}
\`\`\`

## 使用

\`\`\`typescript
import { createHttpClient } from '@company/api-client-core'
import { Api } from '${config.packageName}'

const httpClient = createHttpClient({
  baseURL: 'https://api.example.com'
})

const api = new Api(httpClient)

// 调用 API
const data = await api.getSomething()
\`\`\`

## 开发

\`\`\`bash
# 安装依赖
npm install

# 生成代码
npm run generate

# 运行测试
npm test

# 构建包
npm run build
\`\`\`

## 发布新版本

\`\`\`bash
# 打 Tag
git tag v1.0.1
git push origin v1.0.1

# CI/CD 将自动构建并发布到 NPM
\`\`\`

## License

MIT
`
}
```

---

## 八、最佳实践

### 8.1 OpenAPI 规范管理

#### 规范文件组织
```yaml
# openapi/user-api.yaml
openapi: 3.0.0
info:
  title: User API
  version: 1.0.0
  description: 用户服务 API

servers:
  - url: https://api.example.com/v1
    description: 生产环境
  - url: https://api-staging.example.com/v1
    description: 测试环境

paths:
  /users/{userId}:
    get:
      summary: 获取用户信息
      operationId: getUserProfile
      tags: [User]
      parameters:
        - name: userId
          in: path
          required: true
          schema:
            type: string
      responses:
        '200':
          description: 成功
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'

components:
  schemas:
    User:
      type: object
      properties:
        id:
          type: string
        name:
          type: string
        email:
          type: string
          format: email
```

#### 规范验证
```typescript
// scripts/validate-spec.ts
import SwaggerParser from '@apidevtools/swagger-parser'
import chalk from 'chalk'

async function validateSpec() {
  try {
    const api = await SwaggerParser.validate('./openapi/user-api.yaml')
    console.log(chalk.green('✓ OpenAPI 规范验证通过'))
    console.log(`API: ${api.info.title} v${api.info.version}`)
  } catch (error) {
    console.error(chalk.red('✗ OpenAPI 规范验证失败:'))
    console.error(error.message)
    process.exit(1)
  }
}

validateSpec()
```

### 8.2 版本管理策略

#### 语义化版本规则
- **Major (1.0.0 → 2.0.0)**: Breaking Changes
  - 删除 API 端点
  - 修改请求/响应结构
  - 修改参数类型

- **Minor (1.0.0 → 1.1.0)**: 新功能（向后兼容）
  - 新增 API 端点
  - 新增可选参数
  - 新增响应字段

- **Patch (1.0.0 → 1.0.1)**: Bug 修复
  - 修复错误
  - 性能优化
  - 文档更新

#### Commit 规范
使用 Conventional Commits 规范：

```bash
feat: add user profile endpoint         # 新功能 → Minor
fix: correct email validation           # 修复 → Patch
feat!: remove deprecated user endpoint  # 破坏性变更 → Major
docs: update API documentation          # 文档 → Patch
```

### 8.3 测试策略

```typescript
// tests/integration/user-api.test.ts
import { describe, it, expect, beforeAll, afterAll } from 'vitest'
import { setupServer } from 'msw/node'
import { rest } from 'msw'
import { createHttpClient } from '@company/api-client-core'
import { UserApi } from '../../src/generated/api'

const server = setupServer(
  rest.get('https://api.example.com/users/:userId', (req, res, ctx) => {
    return res(
      ctx.json({
        id: req.params.userId,
        name: 'John Doe',
        email: 'john@example.com'
      })
    )
  })
)

beforeAll(() => server.listen())
afterAll(() => server.close())

describe('UserApi', () => {
  it('should fetch user profile', async () => {
    const httpClient = createHttpClient({
      baseURL: 'https://api.example.com'
    })
    const userApi = new UserApi(httpClient)
    
    const user = await userApi.getUserProfile({ userId: '123' })
    
    expect(user.id).toBe('123')
    expect(user.name).toBe('John Doe')
    expect(user.email).toBe('john@example.com')
  })
})
```

### 8.4 文档管理

#### 自动生成文档
```json
{
  "scripts": {
    "docs:generate": "typedoc src/index.ts --out docs",
    "docs:serve": "http-server docs"
  }
}
```

#### README 模板
每个项目应包含：
- 安装说明
- 快速开始
- API 参考
- 配置选项
- 常见问题
- 变更日志链接

---

## 九、监控和维护

### 9.1 包使用监控

使用 NPM 统计信息监控包的使用情况：

```bash
# 查看包下载量
npm info @company/api-client-user downloads

# 使用第三方工具
npx npm-stat @company/api-client-user
```

### 9.2 依赖更新

定期更新依赖：

```bash
# 检查过期依赖
npm outdated

# 更新依赖
npm update

# 使用工具自动更新
npx npm-check-updates -u
npm install
```

### 9.3 安全审计

```bash
# 运行安全审计
npm audit

# 自动修复漏洞
npm audit fix
```

---

## 十、常见问题 FAQ

### Q1: 如何处理多个 OpenAPI 文件？
**A**: 可以将多个 YAML 文件合并，或者在生成脚本中分别处理：

```typescript
// scripts/generate.ts
import { generateApi } from 'swagger-typescript-api'

const specs = [
  './openapi/user-api.yaml',
  './openapi/admin-api.yaml'
]

for (const spec of specs) {
  await generateApi({
    name: path.basename(spec, '.yaml'),
    input: spec,
    output: './src/generated'
  })
}
```

### Q2: 如何在本地测试 NPM 包？
**A**: 使用 `npm link`：

```bash
# 在 API 客户端项目中
cd api-client-user
npm link

# 在前端应用中
cd my-app
npm link @company/api-client-user

# 测试完成后解除链接
npm unlink @company/api-client-user
```

### Q3: 如何回滚版本？
**A**: 使用 NPM 的 dist-tag：

```bash
# 查看所有版本
npm dist-tag ls @company/api-client-user

# 将 latest 标签指向旧版本
npm dist-tag add @company/api-client-user@1.0.0 latest

# 或者发布一个新的修复版本
```

### Q4: 如何处理敏感信息（如 NPM Token）？
**A**: 使用 GitHub Secrets：

1. 在 NPM 生成 Token: https://www.npmjs.com/settings/tokens
2. 在 GitHub 仓库设置中添加 Secret: `NPM_TOKEN`
3. 在 Actions 中使用: `${{ secrets.NPM_TOKEN }}`

### Q5: 如何支持私有 NPM Registry？
**A**: 修改 `.npmrc` 和 CI/CD 配置：

```ini
# .npmrc
registry=https://npm.company.com/
@company:registry=https://npm.company.com/
```

```yaml
# .github/workflows/publish.yml
- name: Setup Node.js
  uses: actions/setup-node@v4
  with:
    registry-url: 'https://npm.company.com'
```

---

## 十一、迁移指南

### 从单包架构迁移到多包架构

#### 步骤 1: 评估现有 API
```bash
# 列出所有 API 端点
grep -r "operationId" specs/*.yaml | wc -l

# 按服务分组
# user-related: 50 个端点
# order-related: 30 个端点
# payment-related: 20 个端点
```

#### 步骤 2: 创建独立项目
```bash
# 使用脚手架创建各项目
npx @company/create-api-client  # user
npx @company/create-api-client  # order
npx @company/create-api-client  # payment
```

#### 步骤 3: 迁移 OpenAPI 规范
```bash
# 拆分原有的 OpenAPI 文件
# 将 user 相关的端点提取到 user-api.yaml
# 将 order 相关的端点提取到 order-api.yaml
# ...
```

#### 步骤 4: 更新前端应用
```typescript
// 旧方式
import { apiClient } from '@company/api-client'
apiClient.user.getProfile()
apiClient.order.list()

// 新方式
import { userApi } from '@company/api-client-user'
import { orderApi } from '@company/api-client-order'
userApi.getProfile()
orderApi.list()
```

#### 步骤 5: 渐进式迁移
- 保留旧包一段时间，标记为 deprecated
- 逐步迁移各个前端应用
- 监控旧包的使用情况
- 确认无依赖后，下线旧包

---

## 十二、总结

### 核心优势
1. **项目独立**: 各团队独立维护，互不干扰
2. **自动化**: 从代码生成到发布全流程自动化
3. **标准化**: 脚手架统一项目结构和规范
4. **灵活性**: 按需引入，独立升级
5. **可扩展**: 支持多种框架和工具

### 关键要点
- ✅ 使用脚手架快速创建标准化项目
- ✅ 通过 Git Tag 触发自动发布
- ✅ 各项目独立版本管理
- ✅ 公共代码抽取到核心层
- ✅ 完善的 CI/CD 流程

### 成功因素
- 完善的脚手架工具
- 自动化的 CI/CD 流程
- 清晰的版本管理策略
- 良好的文档和示例
- 团队的配合和反馈

---

**文档版本**: 1.0  
**最后更新**: 2025-10-30  
**作者**: API Client Team
