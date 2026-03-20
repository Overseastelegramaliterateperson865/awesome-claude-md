# CLAUDE.md - TypeScript 项目规范

## 语言与编译

- 使用 TypeScript strict 模式，`tsconfig.json` 中 `"strict": true` 必须开启
- 禁止使用 `any`，用 `unknown` 替代后通过类型守卫收窄；确实无法避免时用 `// eslint-disable-next-line @typescript-eslint/no-explicit-any` 并附注释说明原因
- 使用 ESM 模块系统，`"type": "module"` 写在 `package.json` 中，`tsconfig.json` 设置 `"module": "ESNext"`, `"moduleResolution": "bundler"`
- 目标产物 `"target": "ES2022"`，充分利用 `structuredClone`、`Array.at()` 等现代 API

## 代码风格与 Lint

- 优先使用 Biome (`biome.json`) 做格式化和 lint；若项目用 ESLint，则配合 `@typescript-eslint/recommended-type-checked`
- 缩进 2 空格，行尾无分号，单引号，尾逗号 `all`
- 文件命名：`kebab-case.ts`，类型定义文件 `*.types.ts`，常量文件 `*.constants.ts`

## 导入规范

- 导入顺序：node 内置模块 > 第三方库 > 项目内部模块 > 相对路径模块，组间空行分隔
- 使用 `import type { Foo }` 语法导入纯类型，避免运行时引入
- 路径别名统一用 `@/` 指向 `src/`，在 `tsconfig.json` 的 `paths` 中配置

## 类型编写

- 优先用 `interface` 描述对象结构，用 `type` 处理联合类型、交叉类型、工具类型
- 泛型参数命名有意义：`TItem` 而非 `T`，`TResponse` 而非 `R`
- 对外暴露的函数必须显式标注返回类型，内部函数可依赖推断
- 枚举用 `as const` 对象 + `typeof` 替代 `enum`，避免运行时开销

## 测试

- 测试框架使用 Vitest，配置文件 `vitest.config.ts`
- 测试文件放在 `__tests__/` 目录或与源文件同级命名为 `*.test.ts`
- 运行测试：`pnpm test`；运行单个文件：`pnpm test src/utils/__tests__/foo.test.ts`
- 使用 `describe` + `it` 组织用例，描述用英文动词短语，如 `it('returns null when input is empty')`
- Mock 优先用 `vi.fn()` 和 `vi.spyOn()`，避免 mock 整个模块

## 常用命令

- `pnpm dev` - 启动开发服务
- `pnpm build` - 构建产物
- `pnpm lint` - 运行 lint 检查
- `pnpm format` - 格式化代码
- `pnpm typecheck` - 运行 `tsc --noEmit` 类型检查
