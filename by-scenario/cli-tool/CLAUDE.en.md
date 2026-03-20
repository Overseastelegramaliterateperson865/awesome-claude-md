# CLI Tool Project Conventions

## Project Structure

- `src/cli.ts` - Entry point for argument parsing and command routing; `src/commands/` - Subcommand handlers
- `src/utils/` - Utility functions; `bin/` - Executable entry point, referenced by `bin` in `package.json`

## Argument Parsing

- Use `commander` or `yargs` for argument parsing — manual `process.argv` parsing is forbidden
- All commands must support `--help` and `--version`; boolean negation uses `--no-xxx`
- Required arguments use positional arguments; optional configuration uses named options
- Support config files (`.xxxrc.json`) as a fallback for command-line arguments

## Exit Code Conventions

- `0` - Successful execution
- `1` - General error (invalid arguments, runtime exceptions)
- `2` - User-initiated cancellation (Ctrl+C / interactive cancel)
- All non-zero exits must output an error reason to stderr to support script piping

## Output & Logging

- Normal results go to stdout for piping and redirection
- Errors, warnings, and progress indicators go to stderr
- Support a `--json` flag for machine-readable JSON output
- Support `--quiet` / `--verbose` to control log verbosity
- Terminal colors via `chalk` with `NO_COLOR` environment variable detection

## Cross-Platform Compatibility

- File paths: use `path.join()` / `path.resolve()` — never hardcode `/` or `\\`
- Line endings: use `os.EOL`; be mindful of line endings when writing files
- Subprocess calls: use `cross-spawn` instead of native `child_process.spawn`
- Config directory: follow XDG spec — prefer `$XDG_CONFIG_HOME`, fallback to `~/.config/`
- Declare `engines.node` minimum version requirement in `package.json`

## Testing & Publishing

- Every command handler must have unit tests with I/O and filesystem mocked out
- Integration tests invoke the compiled binary via `execa` to verify exit codes and output
- Entry script must include shebang `#!/usr/bin/env node`; run `npm pack --dry-run` before publishing to inspect package contents
