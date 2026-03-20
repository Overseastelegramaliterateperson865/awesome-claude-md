# uni-app 跨端项目

## 技术栈
- uni-app + Vue 3 Composition API + TypeScript
- 目标平台：微信小程序、H5、App（Android/iOS）
- 状态管理：Pinia
- UI 框架：uni-ui 或 uView Plus

## 项目结构
```
src/
  pages/         # 页面，须在 pages.json 中注册
  components/    # 可复用组件（easycom 自动导入）
  composables/   # 组合式函数
  stores/        # Pinia store
  static/        # 静态资源（会被打包进产物）
  uni_modules/   # uni-app 插件模块
pages.json       # 路由和页面配置（核心配置文件）
manifest.json    # 应用配置（appid、权限、SDK 配置等）
```

## 条件编译
- 使用 `#ifdef` / `#ifndef` / `#endif` 实现平台差异化代码
- JS 中：`// #ifdef MP-WEIXIN` ... `// #endif`
- 模板中：`<!-- #ifdef H5 -->` ... `<!-- #endif -->`
- CSS 中：`/* #ifdef APP-PLUS */` ... `/* #endif */`
- 常用平台标识：`MP-WEIXIN`（微信小程序）、`H5`、`APP-PLUS`（App）

## 开发规范
- 使用 Vue 3 `<script setup lang="ts">` 语法
- 页面生命周期用 uni-app 专用钩子：`onLoad`、`onShow`、`onReady`、`onHide`
- 组件生命周期正常使用 Vue 的 `onMounted`、`onUnmounted` 等
- 路由跳转用 `uni.navigateTo`、`uni.redirectTo`、`uni.switchTab`
- 网络请求用 `uni.request`，或封装统一的请求工具函数
- 存储使用 `uni.setStorageSync` / `uni.getStorageSync`

## 多端兼容注意事项
- 小程序不支持 DOM 操作，禁止使用 `document`、`window` 等浏览器 API
- 小程序包大小限制 2MB（主包），超出需配置分包 `subPackages`
- CSS 用 `rpx` 单位（750rpx = 屏幕宽度），组件名避免与原生组件同名

## HBuilderX vs CLI 开发
- HBuilderX 可视化运行/发行；CLI 推荐 VSCode + uni-app 官方插件
- CLI 创建项目：`npx degit dcloudio/uni-preset-vue#vite-ts`

## 常用命令（CLI 模式）
```bash
npm run dev:mp-weixin   # 开发微信小程序（输出到 dist/dev/mp-weixin）
npm run dev:h5          # 开发 H5
npm run build:mp-weixin # 构建微信小程序
npm run build:h5        # 构建 H5
```

## 常见陷阱
- `pages.json` 修改后必须重新编译才能生效
- 小程序端 `v-html` 不可用，需使用 `rich-text` 组件替代
- 微信小程序自定义组件样式隔离，父组件样式默认不穿透到子组件
- `uni.navigateTo` 打开页面上限 10 层，超出需用 `redirectTo` 或 `reLaunch`
- App 端调试使用 `console.log` 在 HBuilderX 控制台查看，不是浏览器 DevTools
