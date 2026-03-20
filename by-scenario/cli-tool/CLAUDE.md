# CLI 工具项目规范

## 项目结构

- `src/cli.ts` - 入口，参数解析和命令路由；`src/commands/` - 子命令 handler
- `src/utils/` - 工具函数；`bin/` - 可执行入口，`package.json` 的 `bin` 指向此处

## 参数解析

- 使用 `commander` 或 `yargs` 处理参数，禁止手动解析 `process.argv`
- 所有命令必须支持 `--help` 和 `--version`；布尔取反用 `--no-xxx`
- 必填参数用 positional argument，可选配置用 named option
- 支持配置文件（`.xxxrc.json`）作为命令行参数的 fallback

## Exit Code 规范

- `0` - 执行成功
- `1` - 一般性错误（参数错误、运行时异常）
- `2` - 用户主动取消（Ctrl+C / 交互式取消）
- 所有非零退出必须在 stderr 输出错误原因，便于脚本管道组合

## 输出与日志

- 正常结果输出到 stdout，供管道和重定向使用
- 错误信息、警告、进度提示输出到 stderr
- 支持 `--json` 标志输出机器可读的 JSON 格式
- 支持 `--quiet` / `--verbose` 控制日志详细程度
- 终端颜色使用 `chalk` 并检测 `NO_COLOR` 环境变量

## 跨平台兼容

- 文件路径使用 `path.join()` / `path.resolve()`，禁止硬编码 `/` 或 `\\`
- 换行符使用 `os.EOL`，写文件时注意 line ending
- 子进程调用使用 `cross-spawn` 代替原生 `child_process.spawn`
- 配置文件目录遵循 XDG 规范：优先 `$XDG_CONFIG_HOME`，fallback `~/.config/`
- 在 `package.json` 中声明 `engines.node` 最低版本要求

## 测试与发布

- 每个 command handler 必须有单元测试，mock 掉 I/O 和文件系统
- 集成测试通过 `execa` 调用编译后的 binary 验证退出码和输出
- 入口脚本首行含 shebang `#!/usr/bin/env node`，发布前 `npm pack --dry-run` 检查打包内容
