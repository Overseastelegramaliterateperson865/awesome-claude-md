# Open Source Library Project

## Project Purpose
- A developer-facing open source utility library or component library
- Published to a public registry (npm / PyPI / crates.io, etc.)
- Prioritizes API design, documentation quality, and backward compatibility

## Project Structure (npm Package Example)
```
src/              # Source code
  index.ts        # Public API entry point
  core/           # Core implementation
  utils/          # Internal utility functions
tests/            # Test files
docs/             # Documentation or docs site source
examples/         # Usage examples
CHANGELOG.md      # Changelog
LICENSE           # Open source license
```

## Versioning
- Follow Semantic Versioning (SemVer): MAJOR (breaking changes) / MINOR (new features) / PATCH (bug fixes)
- Manage with Changesets or Conventional Commits; update CHANGELOG.md with every release

## API Design Principles
- Minimize the public API surface; use clear, consistent naming; provide sensible defaults (zero-config out of the box)
- Use an options object when there are more than 2 optional parameters; ensure full TypeScript type inference
- For breaking changes, provide a migration guide and codemod when feasible

## Documentation Requirements
- README.md: installation instructions, quick start guide, core API overview
- API docs: parameter descriptions and usage examples for every public function/class
- Use TSDoc/JSDoc comments to enable auto-generated API documentation
- Provide runnable, complete examples in the `examples/` directory

## Testing Strategy
- Unit tests should cover all public APIs and edge cases
- Tests serve as documentation: write clear descriptions that demonstrate usage patterns
- Run tests: `npm test` or `pytest`
- Target coverage > 90%

## CI/CD Pipeline
- PR checks: lint + type-check + test + build
- Release flow: tag trigger -> build -> publish to package registry
- Automated release notes generation

## Common Commands
```bash
npm run lint && npm run test && npm run build  # Full pre-publish validation
npm pack --dry-run    # Inspect what will be published (no extraneous files)
npm publish           # Publish to npm
```

## Common Pitfalls
- Never introduce breaking changes in a patch release — it erodes user trust
- Use the `files` field in `package.json` to control published contents; avoid leaking source or test files
- Place framework dependencies (e.g., React) in `peerDependencies` to prevent duplicate installations
- Maintain `.npmignore` separately from `.gitignore` — published contents differ from repo contents
- Provide both ESM and CJS exports (configure dual format via the `exports` field)
- Before publishing, test installation in a clean environment: `npx create-test-app && npm install your-lib`
