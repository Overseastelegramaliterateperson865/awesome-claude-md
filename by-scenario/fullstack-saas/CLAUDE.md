# Full-Stack SaaS 项目规范

## 项目结构

- `frontend/` - 前端（React/Next.js）；`backend/` - 后端（Node.js/NestJS）
- `shared/` - 共享类型和常量；`database/` - migrations 和 seed 脚本
- `infra/` - 基础设施（Docker、Terraform）；`docker-compose.yml` - 本地环境编排

## 前后端分离规范

- 前后端通过 REST API 或 tRPC 通信，接口定义放 `shared/api-types.ts`
- 前端禁止直接访问数据库或依赖后端内部模块
- API 响应格式统一：`{ code: number, data: T, message: string }`
- 错误码前后端共享，定义在 `shared/error-codes.ts`
- 前端使用 `react-query` / `swr` 管理服务端状态，禁止在组件内裸调 fetch

## 认证与授权

- 使用 JWT access token（短期，15min）+ refresh token（长期，7d）双 token 模式
- access token 放 Authorization header，refresh token 放 httpOnly cookie
- 后端每个路由通过 middleware 校验权限，格式：`requireRole('admin')`
- 多租户隔离：所有数据库查询必须带 `tenant_id` 条件，ORM 层统一注入
- OAuth 第三方登录（Google/GitHub）回调处理放 `backend/src/auth/providers/`

## 数据库与 Migration

- 使用 Prisma 或 Drizzle 作为 ORM，schema 即文档
- 每次 schema 变更必须生成 migration 文件：`npx prisma migrate dev --name <描述>`
- migration 文件禁止手动修改已提交的历史记录，只能追加新 migration
- 表名使用 snake_case 复数形式（`users`、`subscription_plans`）
- 必须包含 `created_at`、`updated_at` 字段，软删除加 `deleted_at`
- seed 脚本提供开发环境基础数据，执行命令：`pnpm db:seed`

## 环境变量管理

- 每个服务维护 `.env.example` 列出所有需要的环境变量（不含实际值）
- 敏感变量（API key、DB 密码）禁止提交到代码仓库
- 前端环境变量以 `NEXT_PUBLIC_` 开头，只暴露非敏感配置
- 后端启动时使用 `zod` 校验环境变量完整性，缺失则立即报错退出
- 多环境配置：`.env.development` / `.env.staging` / `.env.production`

## 订阅与计费

- 集成 Stripe 处理支付，webhook 端点验证签名后处理事件
- 用户套餐信息缓存在本地数据库，通过 webhook 同步，不实时查 Stripe
- 功能限制（rate limit、配额）在后端 middleware 层统一拦截
- 免费套餐的限制写在配置文件中，方便调整无需改代码

## 部署规范

- 前端 Vercel / Cloudflare Pages，后端 Railway / Fly.io，数据库用托管服务
- CI：lint -> type-check -> test -> build -> deploy
- staging 与 production 配置隔离，发布前在 staging 验证 migration

## 常用命令

- `pnpm dev` - 同时启动前后端（concurrently）
- `pnpm db:migrate` - 数据库迁移；`pnpm db:studio` - 数据库可视化
- `pnpm test` - 全部测试；`pnpm typecheck` - 类型检查
