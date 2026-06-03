# InfinityBlue API Docs

OpenAPI 3.1 specification + Mintlify documentation site for
[InfinityBlue API](https://infinityblue.com) — a unified AI API
gateway exposing chat, image, video, and model discovery endpoints.

**Live site**: <https://inbllc.mintlify.app> (default: 中文)
**GitHub**: <https://github.com/yzlltyyh/infinityblue-api-docs>

## Repository layout

```
openapi/                 OpenAPI 3.1 specification (source of truth)
├── openapi.yaml         Entry point
├── openapi.bundled.yaml Generated publish artifact (Mintlify reads this)
├── paths/               Per-endpoint YAML
└── components/          Schemas + shared error responses
docs.json                Mintlify config (navigation, theme, OpenAPI pointer)
index.mdx                English root page
introduction.mdx         English introduction
quickstart/              English quickstart (auth, first-request, errors)
api-reference/           English API reference intros (chat, images, videos, models)
guides/                  English in-depth guides (9 topics)
zh/                      Chinese mirror of all MDX content
favicon.svg              Repo-root favicon (docs.json references /favicon.svg)
.mintignore              Excludes source OpenAPI from Mintlify's auto-scan
AGENTS.md                AI agent conventions
package.json             npm scripts
redocly.yaml             Redocly lint config
images/                  Image assets
```

## Local development

```bash
npm install
npm run dev          # Mintlify local preview at http://localhost:3000
```

## Validation

```bash
npm run lint         # Redocly CLI — fast local schema check
npm run validate     # Mintlify validate — same engine as deploy
npm run bundle       # Regenerate openapi/openapi.bundled.yaml
```

The bundled file is what Mintlify reads (`docs.json` points to it).
Mintlify does **not** support cross-file `$ref`, so bundling is
required after any change to source OpenAPI files. See
[`AGENTS.md`](AGENTS.md) for the full workflow and rationale.

## Deployment

Auto-deploys to Mintlify on every `git push` to `main`. No CI/CD
required. Takes 1-2 minutes from push to live.

## Languages

Default is **中文 (zh)**. Language switcher in the top-right of the
docs site lets users toggle to **English**. Both languages have
complete content; translation is light-touch (adapt, don't literally
translate).

## For AI agents

This repository is designed to be modified by AI agents
(Cursor, Claude Code, Windsurf). The complete convention document is
[`AGENTS.md`](AGENTS.md) — read it first before editing anything.

## Integrations

- **Apifox** (debugging sandbox): consumer of our OpenAPI via scheduled
  URL import. See `guides/apifox-sandbox.mdx`.
- **Mintlify MCP** (agent-readable docs): Search + Admin MCP servers
  expose the live docs to IDE agents. See `guides/mcp-integration.mdx`.

## Endpoints exposed (13 total)

- **Models**: `GET /v1/models`, `GET /v1beta/models`
- **Chat**: `POST /v1/chat/completions`, `POST /v1/responses`,
  `POST /v1/messages`, `POST /v1beta/models/{model}:generateContent`
- **Images**: `POST /v1/images/generations`, `POST /v1/images/edits`,
  `POST /v1beta/models/{model}:generateContent`
- **Videos**: `POST /v1/videos`, `GET /v1/videos/{task_id}`,
  `GET /v1/videos/{task_id}/content`
- **Legacy** (deprecated): `POST /v1/video/generations`,
  `GET /v1/video/generations/{task_id}`

`audio`, `embeddings`, `rerank` are not on offer yet — placeholder
files exist in `api-reference/` but are not in the navigation.
