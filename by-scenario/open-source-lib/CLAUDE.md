# 开源库项目

## 项目定位
- 面向开发者的开源工具库/组件库
- 发布到 npm / PyPI / crates.io 等公共仓库
- 重视 API 设计、文档质量、向后兼容性

## 项目结构（以 npm 包为例）
```
src/              # 源代码
  index.ts        # 公开 API 入口
  core/           # 核心实现
  utils/          # 内部工具函数
tests/            # 测试文件
docs/             # 文档或文档站源码
examples/         # 使用示例
CHANGELOG.md      # 变更日志
LICENSE           # 开源许可证
```

## 版本管理
- 语义化版本（SemVer）：MAJOR（Breaking Change）/ MINOR（新功能）/ PATCH（修复）
- 使用 Changesets 或 Conventional Commits 管理，每次发版更新 CHANGELOG.md

## API 设计原则
- 最小化公开 API 面积，命名清晰一致，合理的默认值（零配置开箱使用）
- 超过 2 个可选参数时用 options 对象，确保 TypeScript 完整类型推导
- Breaking Change 提供迁移指南和 codemod（如可能）

## 文档要求
- README.md：安装方式、快速上手、核心 API 概览
- API 文档：每个公开函数/类的参数说明和使用示例
- 使用 TSDoc/JSDoc 注释，可自动生成 API 文档
- examples/ 目录提供可运行的完整示例

## 测试策略
- 单元测试覆盖所有公开 API 和边界情况
- 测试即文档：测试用例描述清晰，体现使用方式
- 运行测试：`npm test` 或 `pytest`
- 目标覆盖率 > 90%

## CI/CD 流程
- PR 检查：lint + type-check + test + build
- 发版流程：tag 触发 -> 构建 -> 发布到包管理器
- 自动化 release notes 生成

## 常用命令
```bash
npm run lint && npm run test && npm run build  # 发布前全量检查
npm pack --dry-run    # 检查发布内容（无多余文件）
npm publish           # 发布到 npm
```

## 常见陷阱
- 不要在补丁版本中引入 Breaking Change，会破坏用户信任
- `package.json` 的 `files` 字段控制发布内容，避免泄漏源码或测试文件
- 依赖尽量放 `peerDependencies`（如 React），避免重复安装
- `.npmignore` 与 `.gitignore` 独立维护，发布内容和仓库内容不同
- 导出格式同时提供 ESM 和 CJS（`exports` 字段配置双格式）
- 发布前在干净环境测试安装：`npx create-test-app && npm install your-lib`
