# CLAUDE.md - Go Project Conventions

## Build & Run

- Use Go Modules for dependency management; run `go mod tidy` to keep go.sum clean
- Build: `go build ./cmd/...`; Test: `go test ./... -race -count=1`
- Lint: `golangci-lint run` — must produce zero warnings

## Project Structure

- `cmd/` - Executable entry points, each subdirectory contains a `main.go`
- `internal/` - Private business logic, cannot be imported by external projects
- `pkg/` - Public library code that may be imported externally
- `api/` - Protobuf / OpenAPI definition files
- `configs/` - Configuration file templates (never commit actual secrets)

## Error Handling

- Never use `panic` in library code; `log.Fatal` is only acceptable during `main` initialization
- Wrap errors with `fmt.Errorf("operation description: %w", err)` to preserve the error chain
- Custom error types must implement the `Error()` interface; implement `Unwrap()` when needed
- Handle errors at the call site uniformly — avoid duplicate logging

## Interface Design

- Define interfaces on the consumer side, not the implementation side (accept interfaces, return structs)
- Keep interfaces small and focused — prefer single-method interfaces (e.g., `io.Reader`)
- Use interfaces for dependency injection to facilitate mock testing

## Testing Conventions

- Unit test files live alongside source code, named `xxx_test.go`
- Use `testify/assert` and `testify/require` for concise assertions
- Use table-driven tests for multiple scenarios, with subtests via `t.Run("scenario name", ...)`
- Integration tests use a `//go:build integration` build tag; run separately in CI
- Mocking: use `testify/mock` or `gomock`, generated at the interface definition site

## Code Style

- Strictly follow `gofmt` formatting; run `goimports` before committing
- Exported functions and types must have GoDoc comments starting with the function name
- Pass `context.Context` as the first parameter, named `ctx`
- Do not use `init()` functions — explicit initialization is preferred over implicit
- Channels and goroutines must have a clear exit mechanism to prevent leaks
