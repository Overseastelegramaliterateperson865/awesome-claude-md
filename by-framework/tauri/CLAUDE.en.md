# Tauri Desktop Application Project

## Tech Stack
- Tauri 2 framework: Rust backend + Web frontend
- Frontend: any framework (React/Vue/Svelte/plain HTML)
- Rust: handles system-level operations, file I/O, and native functionality
- Build tools: Cargo (Rust) + frontend bundler (Vite, etc.)

## Project Structure
```
src-tauri/
  src/
    main.rs      # Application entry point — window and plugin configuration
    lib.rs       # Tauri command definitions
  Cargo.toml     # Rust dependencies
  tauri.conf.json # Tauri config (window size, permissions, bundling, etc.)
  capabilities/  # Permission declaration files
src/             # Frontend code directory
```

## Tauri Commands (Rust <-> JS Communication)
- Define commands in Rust with the `#[tauri::command]` macro; register them in `main.rs` via `invoke_handler![cmd1, cmd2]`
- Frontend: `import { invoke } from '@tauri-apps/api/core'`, then `await invoke('cmd_name', { arg })`
- Parameters are automatically serialized/deserialized (Rust side uses serde); command names follow snake_case

## Event System
- Frontend to backend: `emit('event-name', payload)`
- Backend to frontend: `app_handle.emit("event-name", payload)`
- Frontend listener: `listen('event-name', (event) => { ... })`
- Best suited for backend-initiated data pushes (e.g., progress updates, file watcher callbacks)

## Rust Development Guidelines
- Handle errors with `Result<T, E>`; commands should return `Result<T, String>` or a custom error type
- Use `std::fs` for file operations and `tokio` for async work
- Manage application state with `tauri::Manager`'s `manage()`; access it in commands via `State<>`
- Sensitive operations (filesystem, shell, etc.) require permission declarations in `capabilities/`

## Frontend Development Guidelines
- Use `@tauri-apps/api` to access system APIs; check `window.__TAURI_INTERNALS__` to detect the runtime environment
- Window operations: `@tauri-apps/api/window`; file dialogs: `@tauri-apps/plugin-dialog`

## Common Commands
```bash
npm run tauri dev      # Start dev mode (launches both frontend and Rust backend)
npm run tauri build    # Build for production (generates installer packages)
cargo test             # Run Rust unit tests (from src-tauri/)
cargo clippy           # Run Rust linter
```

## Building and Packaging
- macOS produces `.dmg`, Windows produces `.msi`/`.exe`, Linux produces `.deb`/`.AppImage`
- Cross-compilation requires CI setup (official GitHub Actions templates are available)
- Auto-update support via `@tauri-apps/plugin-updater`; requires a signing key configuration

## Common Pitfalls
- Tauri 2 APIs are not compatible with Tauri 1 — make sure you are reading the correct docs
- First Rust compilation is slow (downloading + compiling dependencies); subsequent incremental builds are fast
- The `identifier` in `tauri.conf.json` must be unique (reverse domain name format)
- Frontend asset paths need the `asset:` protocol or `convertFileSrc()` to reference local files
- Watch out for path separator differences on Windows; use `std::path::PathBuf` in Rust
