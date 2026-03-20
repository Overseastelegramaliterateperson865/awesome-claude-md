# uni-app Cross-Platform Project

## Tech Stack
- uni-app + Vue 3 Composition API + TypeScript
- Target platforms: WeChat Mini Program, H5 (web), App (Android/iOS)
- State management: Pinia
- UI framework: uni-ui or uView Plus

## Project Structure
```
src/
  pages/         # Pages (must be registered in pages.json)
  components/    # Reusable components (auto-imported via easycom)
  composables/   # Composable functions
  stores/        # Pinia stores
  static/        # Static assets (bundled into the output)
  uni_modules/   # uni-app plugin modules
pages.json       # Route and page configuration (core config file)
manifest.json    # App configuration (appid, permissions, SDK settings, etc.)
```

## Conditional Compilation
- Use `#ifdef` / `#ifndef` / `#endif` directives for platform-specific code
- In JS: `// #ifdef MP-WEIXIN` ... `// #endif`
- In templates: `<!-- #ifdef H5 -->` ... `<!-- #endif -->`
- In CSS: `/* #ifdef APP-PLUS */` ... `/* #endif */`
- Common platform identifiers: `MP-WEIXIN` (WeChat Mini Program), `H5`, `APP-PLUS` (native App)

## Development Guidelines
- Use Vue 3 `<script setup lang="ts">` syntax
- For page lifecycle, use uni-app specific hooks: `onLoad`, `onShow`, `onReady`, `onHide`
- For component lifecycle, use standard Vue hooks: `onMounted`, `onUnmounted`, etc.
- Navigation: `uni.navigateTo`, `uni.redirectTo`, `uni.switchTab`
- Network requests: `uni.request`, or wrap it in a unified request utility
- Storage: `uni.setStorageSync` / `uni.getStorageSync`

## Cross-Platform Compatibility Notes
- Mini Programs do not support DOM manipulation — never use `document`, `window`, or other browser APIs
- Mini Program main package size limit is 2MB; use `subPackages` for anything beyond that
- Use `rpx` units for CSS (750rpx = screen width); avoid naming components the same as native components

## HBuilderX vs CLI Development
- HBuilderX provides a visual run/build workflow; for CLI, use VSCode with the official uni-app extension
- Create a project via CLI: `npx degit dcloudio/uni-preset-vue#vite-ts`

## Common Commands (CLI Mode)
```bash
npm run dev:mp-weixin   # Dev mode for WeChat Mini Program (output: dist/dev/mp-weixin)
npm run dev:h5          # Dev mode for H5
npm run build:mp-weixin # Production build for WeChat Mini Program
npm run build:h5        # Production build for H5
```

## Common Pitfalls
- Changes to `pages.json` require a recompilation to take effect
- `v-html` is not available in Mini Programs — use the `rich-text` component instead
- WeChat Mini Program custom components have style isolation; parent styles do not penetrate child components by default
- `uni.navigateTo` has a 10-page stack limit; use `redirectTo` or `reLaunch` to avoid hitting it
- On the App platform, use `console.log` and view output in the HBuilderX console, not browser DevTools
