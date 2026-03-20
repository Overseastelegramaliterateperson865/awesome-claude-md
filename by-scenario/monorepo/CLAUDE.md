# Monorepo 项目规范 (Turborepo + pnpm workspace)

## 工作区结构

- `apps/` - 可部署应用（web、api、admin 等）
- `packages/` - 共享包（ui、utils、config、tsconfig 等）
- `tooling/` - 开发工具链配置（eslint-config、prettier-config 等）
- 根目录 `pnpm-workspace.yaml` 定义所有 workspace 成员

## 依赖管理

- 始终使用 `pnpm add` 安装依赖，禁止使用 npm/yarn
- 添加依赖到特定 workspace：`pnpm add <pkg> --filter <workspace>`
- 共享包之间通过 `"@repo/ui": "workspace:*"` 引用，禁止写固定版本号
- 根目录 `package.json` 只放 devDependencies（husky、lint-staged 等）

## 构建顺序与缓存

- Turborepo 根据 `turbo.json` 中的 `dependsOn` 自动推断构建拓扑顺序
- `packages/` 先于 `apps/` 构建，修改共享包后必须重新构建下游消费者
- 利用 `turbo run build --filter=<app>...` 只构建目标及其依赖链
- 远程缓存开启时，CI 中未变更的包会命中缓存跳过构建

## 共享包开发规范

- 每个 `packages/*` 必须有独立的 `package.json`、`tsconfig.json` 和 `src/index.ts` 入口
- 共享 UI 组件放 `packages/ui`，工具函数放 `packages/utils`
- 共享包导出必须在 `package.json` 的 `exports` 字段中显式声明
- 类型定义随源码一起发布，不要单独维护 `.d.ts`

## 测试策略

- 单元测试：各 workspace 独立运行 `pnpm test --filter=<workspace>`
- 集成测试：放在对应 app 目录下，可依赖已构建的共享包
- CI 中使用 `turbo run test --filter=...[origin/main]` 只测试受影响的包
- 每个共享包的测试必须独立通过，不依赖其他 workspace 的运行时状态

## 常用命令

- `pnpm dev --filter=web` - 启动单个应用开发服务器
- `pnpm build` - 全量构建（Turborepo 自动并行+缓存）
- `pnpm lint` - 全量 lint 检查
- `turbo run build --dry` - 查看构建依赖图，不实际执行

## 注意事项

- 新增 workspace 后必须执行 `pnpm install` 更新 lockfile
- 禁止跨 workspace 直接 import 相对路径，必须通过包名引用
- 环境变量按 app 隔离，各 app 维护自己的 `.env.example`
- PR 改动共享包时，CI 应自动触发所有消费者的测试
