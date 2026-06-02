# InfinityBlue API 文档项目

这是 InfinityBlue API 的 OpenAPI 规范 + Mintlify 文档的源代码仓库。
**SSOT（Single Source of Truth）= `openapi/openapi.yaml` 及其 `$ref` 子文件**。

## 工作流

1. 修改 `openapi/` 下的 YAML 文件
2. `npm run lint` 验证（Redocly CLI）
3. `npm run validate`（Mintlify 验证 OpenAPI）
4. `npm run dev`（Mintlify 本地预览，可选）
5. `git commit` + `git push` → Mintlify 自动部署

## 文件组织

```
openapi/
├── openapi.yaml              # 入口（< 100 行）
├── paths/                    # 路径定义
│   ├── chat-completions.yaml
│   ├── images-generations.yaml
│   └── ...
└── components/
    ├── schemas/              # 数据模型
    └── responses/            # 公共错误响应
docs.json                     # Mintlify 导航和配置
*.mdx                         # 英文指南（人话引导）
zh/*.mdx                      # 中文 MDX（人话引导）
```

## 写作约定

- **OpenAPI 用英文**（机器友好，可生成 SDK）
- `description` 写完整句子，1-2 句话解释用途和限制
- 每个接口必须有至少 1 个 `example`
- 多模态用 `tags` 区分：`chat` / `images` / `videos` / `audio` / `embeddings` / `rerank` / `models`
- 错误响应统一引用 `components/schemas/error.yaml`
- 鉴权默认用 `bearerAuth`（在 OpenAPI 顶层 `security`）
- 异步接口（视频）在 description 里说明轮询或 webhook 机制
- 用 `oneOf` + `discriminator` 表达多上游格式差异

## 避免

- 不要用 `any` 类型（用具体的 schema）
- 不要省略 `operationId`
- 不要在 OpenAPI 中混入中文
- 不要直接改入口 `openapi.yaml` 的 `paths` 段（用 `$ref` 引用子文件）
- 不要把密钥、token 写进 OpenAPI 任何地方

## 验证命令

```bash
npm install          # 第一次
npm run lint         # Redocly lint
npm run validate     # Mintlify OpenAPI 验证
npm run dev          # 本地预览（localhost:3000）
```
