# InfinityBlue API 文档项目

This repository is the **source of truth** for the InfinityBlue API
documentation site. It holds the OpenAPI specs (Chinese + English),
bilingual MDX guides, and the Mintlify configuration. AI agents
(Cursor, Claude Code, Windsurf) are the primary editors.

- **Live docs**: https://docs.getinfinityblue.com
- **Model list & pricing**: https://api.getinfinityblue.com/pricing
- **GitHub**: https://github.com/yzlltyyh/infinityblue-api-docs

---

## Repository layout

```
.
├── openapi/                    ← OpenAPI 3.1 specs (per language, per category)
│   ├── chat.zh.yaml            ← 聊天: ChatCompletions / Responses / Gemini / Claude
│   ├── chat.en.yaml
│   ├── images.zh.yaml          ← 图像: 生成 / 编辑 / Nano Banana
│   ├── images.en.yaml
│   ├── videos.zh.yaml          ← 视频: 通用 / Seedance 异步
│   ├── videos.en.yaml
│   ├── models.zh.yaml          ← 模型列表: OpenAI / Gemini 格式
│   └── models.en.yaml
├── docs.json                   ← Mintlify config (per-language navigation, OpenAPI pointers)
├── index.mdx / introduction.mdx        ← English landing + overview
├── quickstart/                 ← English: auth, first-request, errors
├── guides/                     ← English: model-selection, streaming, multimodal-input
├── zh/                         ← Chinese mirror of every MDX (index, introduction, quickstart, guides)
├── images/favicon.svg          ← favicon (docs.json references /favicon.svg)
├── redocly.yaml                ← Redocly lint config
├── package.json                ← npm scripts (lint / validate / dev)
└── AGENTS.md                   ← this file
```

There is **no** `openapi/openapi.yaml`, no `paths/`, no `components/`,
and no bundle step. Each language+category spec is a **single
self-contained file** using internal `$ref` only.

---

## Architecture decisions (read before editing)

### 1. Bilingual API reference = two parallel specs

The Apifox doc (the historical source) is in Chinese, and Chinese is the
default site language. To give Chinese users a **fully localized** API
reference (parameter descriptions in Chinese, not bounced to English),
each category has two specs:

- `*.zh.yaml` — Chinese descriptions (**primary / source of truth for content**)
- `*.en.yaml` — English mirror, **identical structure/schemas/examples**;
  only natural-language strings (`description`, `summary`, `x-mint.content`) differ.

`docs.json` points the `zh` language tree at the `.zh.yaml` specs and the
`en` tree at the `.en.yaml` specs. When you change one language's spec,
**mirror the same structural change to the other language**.

### 2. Self-contained single file per spec (no bundling)

Mintlify only supports `$ref` **within a single document**. We keep each
spec as one self-contained file (internal `$ref: '#/components/...'` only).
This matches the Apifox export format, lets Mintlify read the file
directly, and removes the old "forgot to bundle" footgun. **Do not add
cross-file `$ref`.**

### 3. Navigation mirrors the Apifox grouping

`docs.json` groups endpoints by **native format** (聊天 → 原生 OpenAI /
Gemini / Claude；图像 → OpenAI / Nano Banana；视频 → Seedance / 通用),
matching what users saw on the Apifox doc. Each nav group references its
category spec via `"openapi": { "source": "...", "directory": "..." }`.

### 4. `directory` keys avoid cross-spec / cross-language page collisions

Some endpoints share a path across specs (e.g. Nano Banana reuses
`POST /v1/chat/completions` and `POST /v1beta/models/{model}:generateContent`).
To stop Mintlify from generating colliding page files, every group sets a
distinct `directory` per **(language, category)**: `zh-chat`, `en-chat`,
`zh-images`, … Keep this scheme when adding endpoints.

### 5. Chinese is the default language

`docs.json` → `navigation.languages[].default: true` is on the `zh` entry.

---

## Content rules

- **Real models only.** This relay's actual models live at
  https://api.getinfinityblue.com/pricing. Use real IDs in examples:
  chat `gpt-5.4` / `gpt-5.4-mini` / `gpt-5.5` / `gpt-5.3-codex` /
  `gemini-3.1-pro-preview` / `gemini-2.5-pro` / `deepseek-v4-pro`;
  image `gpt-image-2` / `nanobanana` / `nanobanana_pro` / `nanobanana_2`;
  video `doubao-seedance-2-0-260128` / `veo_3_1` / `kling-v2-5-turbo`.
  **Never** use `gpt-4o`, `dall-e-*`, `whisper-*`, `o3`, `claude-*`,
  `text-embedding-*`, `sora-*` — the relay does not offer them.
- **Real base URL** everywhere: `https://api.getinfinityblue.com`.
- **No hard-coded prices.** Pricing changes often; link to the pricing page.
- **OpenAPI 3.1.0.** No `nullable: true` → use `type: [string, "null"]`.
  No `x-apifox-*` keys. Path params need `required: true`.
- **Per operation:** unique `operationId`, `summary`, `tags`, request
  examples, `200` + error responses (`400/401/429/500` → `ErrorResponse`),
  and `x-mint.metadata` + `x-mint.content` for richer rendering.

---

## Workflow (edit → live in ~3 min)

```bash
# 1. Edit the relevant spec(s) — mirror zh and en
openapi/chat.zh.yaml   (and openapi/chat.en.yaml)
# or any *.mdx (mirror zh/ and the English copy)

# 2. Lint (Redocly — strict, catches most issues)
npm run lint            # lints all 8 specs

# 3. (optional) Local preview — see note below
npm run dev             # mint dev, localhost:3000

# 4. Commit + push
git add . && git commit -m "..." && git push

# 5. Mintlify auto-deploys in 1-2 min → docs.getinfinityblue.com
```

> **Known environment issue:** `mint validate` / `mint dev` may crash
> locally with an "Invalid hook call" React error caused by a nested
> React inside `@mintlify/previewing`. This is an install issue, not a
> spec issue. If it bites you, rely on `npm run lint` (Redocly) locally
> and use a Mintlify **preview deployment** (open a PR) as the
> authoritative render/validation.

---

## npm scripts

| Script | Command | Purpose |
|---|---|---|
| `npm run lint` | `redocly lint` (all 8 specs) | Catch OpenAPI errors fast (strict) |
| `npm run validate` | `mint validate` | Mintlify build validation (see env note) |
| `npm run dev` | `mint dev` | Local preview at localhost:3000 |

---

## Common agent tasks

| Task | Steps |
|---|---|
| Edit an endpoint's description/params | Edit `openapi/<cat>.zh.yaml` **and** `<cat>.en.yaml` → `npm run lint` → commit |
| Add an endpoint | Add the operation + schemas to both `<cat>.{zh,en}.yaml` → add a `"METHOD /path"` entry under the right nav group in `docs.json` (both languages) → `npm run lint` |
| Add a new category | Create `<cat>.{zh,en}.yaml` → add a top-level group in `docs.json` for both languages with a distinct `directory` (`zh-<cat>` / `en-<cat>`) → lint |
| Update example models | grep specs + MDX for the old ID → replace with a real ID from the pricing page → lint |
| Add a guide | Create `guides/<name>.mdx` (en) + `zh/guides/<name>.mdx` (zh) → add to both language `pages` arrays in `docs.json` |

---

## What NOT to do

- ❌ Add cross-file `$ref` (Mintlify won't follow it — keep specs self-contained).
- ❌ Put `nullable: true` in a spec (use `type: [x, "null"]`).
- ❌ Use fake/unavailable models or hard-code prices.
- ❌ Change one language's spec/MDX without mirroring the other.
- ❌ Recreate `openapi/openapi.yaml`, `paths/`, `components/`, or a bundle
  step — the architecture is single-file-per-spec now.
- ❌ Reuse a `directory` value across two different (language, category)
  pairs — it causes page collisions.
