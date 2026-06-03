# InfinityBlue API Docs

Bilingual OpenAPI 3.1 specs + Mintlify documentation site for the
**InfinityBlue API** — a unified AI gateway exposing OpenAI-, Gemini-,
and Claude-compatible endpoints for chat, image, video, and model
discovery.

- **Live docs**: <https://docs.getinfinityblue.com> (default: 中文)
- **Model list & pricing**: <https://api.getinfinityblue.com/pricing>
- **GitHub**: <https://github.com/yzlltyyh/infinityblue-api-docs>

## Repository layout

```
openapi/                 Self-contained OpenAPI 3.1 specs (per language, per category)
├── chat.zh.yaml         聊天: ChatCompletions / Responses / Gemini / Claude
├── chat.en.yaml         (English mirror)
├── images.zh.yaml       图像: 生成 / 编辑 / Nano Banana
├── images.en.yaml
├── videos.zh.yaml       视频: 通用 / Seedance 异步
├── videos.en.yaml
├── models.zh.yaml       模型列表: OpenAI / Gemini 格式
└── models.en.yaml
docs.json                Mintlify config (per-language navigation + OpenAPI pointers)
index.mdx / introduction.mdx   English landing + overview
quickstart/              English: auth, first-request, errors
guides/                  English: model-selection, streaming, multimodal-input
zh/                      Chinese mirror of every MDX
images/favicon.svg       Favicon (docs.json references /favicon.svg)
AGENTS.md                AI agent conventions (read first)
package.json             npm scripts
redocly.yaml             Redocly lint config
```

Each spec is a **single self-contained file** (internal `$ref` only).
There is no bundle step and no shared `paths/` or `components/` tree.

## Local development

```bash
npm install
npm run lint         # Redocly — strict schema check on all 8 specs
npm run dev          # Mintlify preview at http://localhost:3000 (see note)
```

> **Note:** `mint dev` / `mint validate` may crash locally with an
> "Invalid hook call" React error (a nested-React install issue inside
> `@mintlify/previewing`, unrelated to the specs). If so, use `npm run
> lint` locally and a Mintlify **preview deployment** (open a PR) for
> the authoritative render. See [`AGENTS.md`](AGENTS.md).

## Bilingual model

Chinese is the default language and the **source of truth for content**.
Each category ships two specs (`*.zh.yaml` + `*.en.yaml`) with identical
structure — only the natural-language strings differ. When you edit one
language, mirror the structural change to the other. Full rationale in
[`AGENTS.md`](AGENTS.md).

## Deployment

Auto-deploys to Mintlify on every `git push` to `main` (1-2 min from
push to live). No CI/CD required.

## Endpoints (16 total)

- **Models**: `GET /v1/models`, `GET /v1beta/models`
- **Chat**: `POST /v1/chat/completions`, `POST /v1/responses`,
  `POST /v1/messages` (Claude), `POST /v1beta/models/{model}:generateContent` (Gemini)
- **Images**: `POST /v1/images/generations`, `POST /v1/images/edits`,
  plus Nano Banana via `:generateContent` and `/v1/chat/completions`
- **Videos**: `POST /v1/videos`, `GET /v1/videos/{task_id}`,
  `GET /v1/videos/{task_id}/content`, `POST /v1/video/generations`,
  `GET /v1/video/generations/{task_id}`

## For AI agents

This repo is designed to be edited by AI agents (Cursor, Claude Code,
Windsurf). Read [`AGENTS.md`](AGENTS.md) before changing anything.
