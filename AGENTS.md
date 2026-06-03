# InfinityBlue API 文档项目

This repository is the **source of truth** for the InfinityBlue API
documentation site. It contains the OpenAPI specification, MDX guides,
and Mintlify configuration. AI agents (Cursor, Claude Code, Windsurf)
are the primary editors.

**Mintlify deployment**: https://inbllc.mintlify.app
**GitHub**: https://github.com/yzlltyyh/infinityblue-api-docs

---

## Repository layout

```
.
├── openapi/                      ← OpenAPI 3.1 spec (SSOT)
│   ├── openapi.yaml              ← Entry (referenced by openapi.yaml docs.json via bundled)
│   ├── openapi.bundled.yaml      ← Generated publish artifact (Mintlify reads this)
│   ├── paths/                    ← Per-endpoint YAML (source of truth)
│   └── components/
│       ├── schemas/              ← Data models
│       └── responses/            ← Shared error responses
├── docs.json                     ← Mintlify config (navigation, theme, OpenAPI pointer)
├── index.mdx                     ← English root
├── introduction.mdx              ← English introduction
├── quickstart/                   ← English quickstart guides
├── api-reference/                ← English API reference intro pages
├── guides/                       ← English in-depth guides
├── zh/                           ← Chinese mirror of all MDX
│   ├── index.mdx
│   ├── introduction.mdx
│   ├── quickstart/
│   ├── api-reference/
│   └── guides/
├── favicon.svg                   ← Repo-root favicon (docs.json references /favicon.svg)
├── .mintignore                   ← Excludes source OpenAPI from Mintlify's auto-scan
├── AGENTS.md                     ← This file
├── README.md                     ← Human-facing overview
├── package.json                  ← npm scripts
├── redocly.yaml                  ← Redocly lint config
└── images/                       ← Image assets
```

---

## Complete workflow (edit → live in ~3 min)

```
1. Edit source file
   ↓
   openapi/paths/foo.yaml  (or components/schemas/bar.yaml, or any .mdx)
   ↓
2. Validate
   ↓
   npm run lint          # Redocly CLI (rules in redocly.yaml)
   npm run validate      # Mintlify validate (same engine as deploy)
   ↓
3. Bundle OpenAPI (only if openapi/ changed)
   ↓
   npm run bundle        # → openapi/openapi.bundled.yaml
   ↓
4. Local preview (optional)
   ↓
   npm install -g mint    # one-time
   mint dev              # localhost:3000, live reload
   ↓
5. Commit + push
   ↓
   git add . && git commit -m "..." && git push
   ↓
6. Mintlify auto-deploys (1-2 min)
   ↓
   docs.your-domain.com
```

---

## Key constraints

### 1. `openapi/openapi.bundled.yaml` is the publish artifact

Mintlify **does not support cross-file `$ref`**. So:
- Source of truth = `openapi/paths/*.yaml` + `openapi/components/**/*.yaml`
- Mintlify reads = `openapi/openapi.bundled.yaml` (single file, all refs inlined)

`docs.json` points to the bundled file. Whenever you change any
source YAML, you must regenerate the bundle:

```bash
npm run bundle
```

`redocly bundle openapi/openapi.yaml --output openapi/openapi.bundled.yaml`
is what `npm run bundle` runs. Add the bundled file to your commit.

### 2. `.mintignore` excludes source OpenAPI from Mintlify's auto-scan

Mintlify scans all YAML files in the repo. If it finds `openapi.yaml`
with external `$ref`, validation fails. `.mintignore` lists the
source files to skip. **Don't remove entries from this file** unless
you also move the source files elsewhere.

### 3. docs.json navigation is per-language trees

Each language under `navigation.languages[]` has its own
`tabs[].groups[].pages[]` array. Pages for non-default languages
**must** be prefixed with the language code (e.g. `zh/quickstart/auth`).

### 4. `favicon.svg` lives at repo root

`docs.json` references `/favicon.svg`. Don't move it to a subdirectory.

---

## Editing conventions

### OpenAPI

- **Use English** in OpenAPI (description, examples, schema names).
  English is machine-friendly and SDK-generation compatible.
- **One endpoint per file** in `openapi/paths/`. Filename matches the
  HTTP verb + path (e.g. `videos-create.yaml` for `POST /v1/videos`).
- **One schema per file** in `openapi/components/schemas/`. Use
  `$ref: ./bar.yaml` to compose.
- **Path parameters must have `required: true`**. OpenAPI 3.1
  requires this; Mintlify validation fails otherwise.
- **Use `oneOf` + `discriminator`** for upstream-format polymorphism
  (e.g. OpenAI vs Anthropic Messages format).
- **Always include at least one `example`** per request body.
- **Use `x-mint` extension** for richer Mintlify rendering:
  ```yaml
  x-mint:
    metadata:
      title: "Friendly title"
      sidebarTitle: "Sidebar title"
      description: "One-line description"
    content: |
      ## Optional extra Markdown content
      shown below the auto-generated reference.
  ```

### MDX guides

- Front matter: `title`, `description` (used for SEO and search)
- Use Mintlify components: `<Card>`, `<CardGroup>`, `<Note>`,
  `<Warning>`, `<Steps>`, `<Step>`, `<Tabs>`, `<Tab>`, `<CodeGroup>`,
  `<AccordionGroup>`, `<Accordion>`, `<Tooltip>`
- Internal links: `[text](path)` or `[text](/zh/quickstart/auth)`
  (use absolute paths in the current language)
- Code blocks: prefer `<CodeGroup>` with language tabs for SDK examples

### Chinese mirror

When you add or change an English MDX file, mirror it to
`zh/<same-path>`. Translation is light-touch — adapt, don't literally
translate, especially for code identifiers.

---

## Tooling reference

### npm scripts (in `package.json`)

| Script | Command | Purpose |
|---|---|---|
| `npm run lint` | `redocly lint openapi/openapi.yaml` | Catch schema errors fast |
| `npm run bundle` | `redocly bundle openapi/openapi.yaml --output openapi/openapi.bundled.yaml` | Generate publish artifact |
| `npm run validate` | `mint validate openapi/openapi.yaml` | Same engine Mintlify uses for deploy validation |
| `npm run dev` | `mint dev` | Local preview at localhost:3000 |

### Mintlify CLI

Install once: `npm install -g mint`

| Command | Purpose |
|---|---|
| `mint dev` | Local dev server, live reload, OpenAPI reference renders live |
| `mint validate` | Same validator as the deploy pipeline |
| `mint build` | Production build (mostly for debugging) |

### Redocly CLI

Install once: `npm install` (in this repo's package.json)

| Command | Purpose |
|---|---|
| `npx @redocly/cli lint <file>` | Catch OpenAPI errors (faster than `mint validate`) |
| `npx @redocly/cli bundle <entry> --output <out>` | Bundle multi-file OpenAPI into one file |

Redocly is more strict than Mintlify's validator. A pass on Redocly
usually means Mintlify will pass too.

### Git workflow

```bash
# Standard edit
git checkout -b feat/some-change
# ... edit files ...
npm run lint
npm run bundle  # if OpenAPI changed
git add .
git commit -m "feat(openapi): add /v1/foo endpoint"
git push origin feat/some-change
# Open PR on GitHub, review, merge
```

---

## External integrations

### Mintlify MCP server

AI agents can directly read your deployed docs:

`~/.cursor/mcp.json`:
```json
{
  "mcpServers": {
    "mintlify-search": {
      "url": "https://docs.mintlify.com/.well-known/mcp/your-project-id"
    }
  }
}
```

See `guides/mcp-integration.mdx` for full setup.

### Apifox debugging sandbox

Apifox is a **consumer** of our OpenAPI. It does NOT push back to Git.
Setup: see `guides/apifox-sandbox.mdx`.

Brief: Apifox project → Data Management → External Data Sources → Add
URL `https://inbllc.mintlify.app/openapi.yaml` → sync hourly.

### llms.txt (auto-generated)

Mintlify auto-exposes:
- `https://inbllc.mintlify.app/llms.txt` (index)
- `https://inbllc.mintlify.app/llms-full.txt` (full content)
- `https://inbllc.mintlify.app/.well-known/skill.md` (agent skill)

ChatGPT, Claude Code, and other agents know to look for these. You
don't need to do anything to enable them.

---

## What NOT to do

- ❌ Edit `openapi/openapi.bundled.yaml` directly — it's generated.
  Edit source files and re-bundle.
- ❌ Put `nullable: true` in OpenAPI — OpenAPI 3.1 deprecated this.
  Use `type: [string, "null"]` instead.
- ❌ Skip the `bundle` step after editing OpenAPI source files.
- ❌ Add cross-file `$ref` to `openapi.yaml` thinking Mintlify will
  follow them. It won't.
- ❌ Edit `index.mdx` in `zh/` (it doesn't exist; mintlify uses
  `zh/index.mdx` for Chinese root).
- ❌ Push before running `npm run lint` and `npm run validate`.
  Fix errors locally first; debugging deploy failures is slower.
- ❌ Remove entries from `.mintignore` without understanding why
  they're there (it prevents Mintlify from scanning source OpenAPI).

---

## Common agent tasks

| Task | Steps |
|---|---|
| Add new endpoint | Create `openapi/paths/<verb-path>.yaml` → add `$ref` in `openapi.yaml` entry → `npm run lint && npm run bundle && npm run validate` → commit bundled + source |
| Update description on existing endpoint | Edit `openapi/paths/<file>.yaml` → `npm run lint && npm run bundle && npm run validate` → commit |
| Add new schema | Create `openapi/components/schemas/<name>.yaml` → reference in `openapi.yaml` entry's `components.schemas` → bundle |
| Add new guide | Create `guides/<name>.mdx` (en) + `zh/guides/<name>.mdx` (zh) → add to both `tabs[].groups[].pages[]` in `docs.json` |
| Rename a model | grep all `paths/` + `components/schemas/` → update all `$ref` + all example values → re-bundle → validate |
| Add a new model category (e.g. embeddings) | Add OpenAPI paths + components → add `zh/api-reference/embeddings.mdx` + `api-reference/embeddings.mdx` → wire into `docs.json` → bundle |
