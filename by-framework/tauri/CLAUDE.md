# Tauri 桌面应用项目

## 技术栈
- Tauri 2 框架：Rust 后端 + Web 前端
- 前端：任意框架（React/Vue/Svelte/纯 HTML）
- Rust：处理系统级操作、文件 IO、原生功能
- 构建工具：Cargo（Rust）+ 前端构建工具（Vite 等）

## 项目结构
```
src-tauri/
  src/
    main.rs      # 应用入口，配置窗口和插件
    lib.rs       # Tauri command 定义
  Cargo.toml     # Rust 依赖
  tauri.conf.json # Tauri 配置（窗口大小、权限、打包等）
  capabilities/  # 权限声明文件
src/             # 前端代码目录
```

## Tauri Commands（Rust <-> JS 通信）
- Rust 端用 `#[tauri::command]` 宏定义，`main.rs` 中 `invoke_handler![cmd1, cmd2]` 注册
- 前端：`import { invoke } from '@tauri-apps/api/core'`，`await invoke('cmd_name', { arg })`
- 参数自动序列化/反序列化（Rust 使用 serde），命令名统一 snake_case

## 事件系统
- 前端到后端：`emit('event-name', payload)`
- 后端到前端：`app_handle.emit("event-name", payload)`
- 前端监听：`listen('event-name', (event) => { ... })`
- 适用于后端主动推送数据（如进度更新、文件监听回调）

## Rust 开发规范
- 使用 `Result<T, E>` 处理错误，command 返回 `Result<T, String>` 或自定义错误类型
- 文件操作用 `std::fs`，异步操作用 `tokio`
- 状态管理用 `tauri::Manager` 的 `manage()` 注入，command 中通过 `State<>` 获取
- 敏感操作（文件系统、Shell 等）需在 `capabilities/` 中声明权限

## 前端开发规范
- 使用 `@tauri-apps/api` 访问系统 API，`window.__TAURI_INTERNALS__` 判断运行环境
- 窗口操作：`@tauri-apps/api/window`，文件对话框：`@tauri-apps/plugin-dialog`

## 常用命令
```bash
npm run tauri dev      # 启动开发模式（同时启动前端和 Rust 后端）
npm run tauri build    # 构建生产包（生成安装包）
cargo test             # 运行 Rust 单元测试（在 src-tauri/ 下）
cargo clippy           # Rust 代码检查
```

## 构建与打包
- macOS 输出 `.dmg`，Windows 输出 `.msi`/`.exe`，Linux 输出 `.deb`/`.AppImage`
- 交叉编译需配置 CI（GitHub Actions 官方模板可用）
- 更新机制使用 `@tauri-apps/plugin-updater`，需配置签名密钥

## 常见陷阱
- Tauri 2 与 Tauri 1 API 不兼容，注意文档版本
- Rust 编译首次较慢（下载+编译依赖），后续增量编译快
- `tauri.conf.json` 中的 `identifier` 必须唯一（反向域名格式）
- 前端资源路径需使用 `asset:` 协议或 `convertFileSrc()` 转换本地文件路径
- Windows 下路径分隔符差异，Rust 中用 `std::path::PathBuf` 处理
