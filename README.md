# InfinityBlue API Docs

OpenAPI specification + Mintlify documentation site for [InfinityBlue API](https://infinityblue.com).

**Source of truth**: `openapi/openapi.yaml` (and `$ref`'d subfiles)

## Documentation Site

https://infinityblue-api-docs.mintlify.app *(URL updates after first Mintlify deploy)*

## Structure

```
openapi/                 OpenAPI 3.1 specification
├── openapi.yaml         Entry point
├── paths/               Endpoint definitions
└── components/          Reusable schemas and responses
docs.json                Mintlify configuration
*.mdx                    English guides
zh/*.mdx                 Chinese guides
AGENTS.md                AI agent conventions
```

## Local Development

```bash
npm install
npm run dev          # Mintlify local preview at http://localhost:3000
```

## Validation

```bash
npm run lint         # Redocly CLI lint
npm run validate     # Mintlify OpenAPI validation
npm run bundle       # Bundle to single file
```

## Deployment

This site auto-deploys to Mintlify on every push to `main`. No CI/CD required.

## AI Agent Workflow

This repository is designed to be modified by AI agents (Cursor, Claude Code, etc.) under the conventions in `AGENTS.md`. The single source of truth is `openapi/openapi.yaml` — agents should add/modify `$ref` subfiles, not edit the entry file's `paths` block directly.
