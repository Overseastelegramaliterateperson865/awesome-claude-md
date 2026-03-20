# Chrome Extension 项目规范（Manifest V3）

## 项目结构

- `manifest.json` - 扩展清单（Manifest V3）
- `src/background/` - Service Worker；`src/content/` - Content Script
- `src/popup/` - 弹出页 UI；`src/options/` - 设置页；`src/shared/` - 共享代码
- `assets/` - 图标（16/32/48/128px）和静态资源

## Service Worker vs Content Script

- Service Worker 是事件驱动的，无法持久运行，禁止依赖全局变量保持状态
- 状态持久化必须用 `chrome.storage`，监听事件在顶层注册
- 网络请求拦截使用 `chrome.declarativeNetRequest`（非 `webRequest`）
- Content Script 运行在隔离 JS 环境但共享页面 DOM，操作前检查元素存在性
- CSS 防污染：使用 Shadow DOM 或高特异性选择器加唯一前缀
- Content Script 禁止直接调用大部分 `chrome.*` API，通过 message passing 委托

## 消息传递（Message Passing）

- 使用 `chrome.runtime.sendMessage` / `onMessage` 在各上下文间通信
- 长连接用 `chrome.runtime.connect` 建立 port
- 消息格式统一：`{ type: string, payload: unknown }`，按 `type` 路由分发
- 异步响应时 `onMessage` listener 必须 `return true`

## Storage API 使用

- `storage.local` 本地存储上限 10MB；`storage.sync` 跨设备同步，单项 8KB / 总量 100KB
- 用户配置用 `sync`（跨设备），缓存数据用 `local`
- 监听变更：`chrome.storage.onChanged.addListener((changes, area) => {})`

## 权限声明

- 只声明最小权限集，优先用 `optional_permissions` 按需授权
- `host_permissions` 精确匹配域名，避免 `<all_urls>`；优先用 `activeTab`
- 权限变更会触发扩展禁用，升级时注意兼容性

## 开发与调试

- 使用 Vite / webpack 打包，配置多 entry（background、content、popup）
- 发布前在 `chrome://extensions` 加载 unpacked 做完整流程测试
