# CLAUDE.md - Vue 3 + Nuxt 3 项目规范

## 项目概述

- 基于 Nuxt 3 + Vue 3，适配国内项目需求（中文 UI、国内 CDN、可能需要兼容微信生态）
- 包管理器使用 pnpm，Node.js >= 18

## 项目结构

- `pages/` - 文件系统路由 | `components/` - 自动导入组件 | `composables/` - 组合式函数（`use*.ts`）
- `stores/` - Pinia 状态管理 | `server/api/` - 服务端接口 | `utils/` - 工具函数
- `plugins/` - Nuxt 插件 | `middleware/` - 路由中间件 | `types/api/` - API 响应类型

## Composition API

- 只使用 `<script setup lang="ts">`，禁止 Options API
- 组件内代码顺序：props/emits > 响应式状态 > computed > watch > 方法 > 生命周期钩子
- 优先用 `ref` 保持一致性，props 用 `defineProps<{ title: string }>()` 配合 `withDefaults`

## 自动导入

- Nuxt 3 自动导入 Vue API（`ref`、`computed`、`watch`）及 `composables/`、`utils/` 的导出，无需手动 import
- 第三方库仍需手动 `import`；自动导入类型在 `.nuxt/imports.d.ts` 中生成

## Pinia 状态管理

- Store 放在 `stores/` 目录，命名 `use*Store.ts`，使用 Setup Store 语法（函数式 `defineStore`）
- 持久化使用 `pinia-plugin-persistedstate`，配置在 `nuxt.config.ts` 中

## UI 组件库

- 使用 Element Plus 或 Ant Design Vue，通过 Nuxt 模块按需导入，禁止全量引入 CSS
- 主题定制通过 CSS 变量覆盖，写在 `assets/styles/variables.scss`
- 表单校验统一使用组件库内置规则

## 数据请求

- 使用 `useFetch` / `useAsyncData` 获取数据，不要裸用 `axios` 或 `fetch`
- 封装统一请求拦截器在 `composables/useRequest.ts`，处理 token 注入、错误提示
- 服务端 API 用 `defineEventHandler`，接口返回格式：`{ code: number; data: T; message: string }`

## 国内项目适配

- 日期用 dayjs + `dayjs/locale/zh-cn`，国际化用 `@nuxtjs/i18n` 默认 `zh-CN`
- 生产部署关闭 Google Fonts 等境外资源加载，在 `nuxt.config.ts` 中配置
- 微信相关功能（公众号、小程序 webview）封装在 `composables/useWechat.ts`

## TypeScript 规范

- 开启 strict 模式，禁止 `any`
- 组件 props 和 emits 必须用 TypeScript 类型定义，不用运行时声明

## 常用命令

- `pnpm dev` - 启动开发服务
- `pnpm build` - 构建生产产物
- `pnpm lint` - 运行 ESLint 检查
- `pnpm typecheck` - 运行 `nuxi typecheck`
