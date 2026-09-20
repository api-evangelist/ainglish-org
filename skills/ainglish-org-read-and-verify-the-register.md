---
generated: '2026-09-19'
method: generated
name: Read and independently verify the register
description: Read the ratified Ainglish constructs and prove, from public bytes alone, that what you read is what
  the project published. No credentials.
api: openapi/ainglish-org-openapi.yml
operations:
- apiIndex
- getRegister
- getRegisterRelease
- getRegisterCanonical
- getChangelog
- getAnchors
- getLanguageReference
- health
source: Grounded in the provider's own /developers write-lifecycle page, AGENTS.md and the seven agent runbooks;
  every operationId verified verbatim in openapi/ainglish-org-openapi.yml.
---

# Read and independently verify the register

Read the ratified Ainglish constructs and prove, from public bytes alone, that what you read is what the project published. No credentials.

## Auth
- None. Every operation here is public (`security: []`). See `authentication/ainglish-org-authentication.yml`.

## Steps
1. **Discover** — `apiIndex` (`GET /api/v1`). It lists every endpoint, the pagination contract, the MCP endpoint and the licence (`CC0-1.0`).
2. **Read the live register** — `getRegister` (`GET /api/v1/register`). `entries[]` carry `slug`, `public_id`, `form`, `english_mapping`, `kind`, `ratified_version`, `deprecated_reason`. Cite a construct by `/register/{public_id}`, never by slug alone.
3. **Pin a release** — `getRegisterRelease` (`GET /api/v1/register.json`) returns the hashed release envelope with its `digest`.
4. **Recompute the digest** — `getRegisterCanonical` (`GET /api/v1/register.canonical`, media type `application/jcs+json`). `sha256` of the exact bytes must equal the release digest (recipe: `sha256(JCS({kind:'ainglish.register', count, entries:[{slug,kind,form,english_mapping,version} sorted by slug]}))`). Send `If-None-Match` on re-reads; a `304` means nothing changed.
5. **Walk the chain** — `getChangelog` (`GET /api/v1/changelog`). Verify each `entry_hash = sha256(JCS({seq, prev_hash, event, slug, version, register_digest, ts}))` and that `prev_hash` links; `verify.ok` is the server's own claim — recompute it.
6. **Check the anchor** — `getAnchors` (`GET /api/v1/anchors`) then `GET /anchor/{version}.ots` for the OpenTimestamps proof of the version you pinned. `health` (`GET /api/v1/health`) publishes the anchoring queue and the deployed `openapi_sha256`.
7. **Optional agent-readable reference** — `getLanguageReference` (`GET /api/v1/register/reference.md`, `text/markdown`, `X-Register-Digest` header).

## Rules
- Unknown query parameters are a `400 unknown_query_parameter`, never ignored. See `errors/ainglish-org-problem-types.yml`.
- Responses are cached 60 s (`Cache-Control: max-age=60`); the canonical bytes are immutable per version.
- Do not scrape the site for a corpus — use a release bundle (`https://ainglish.org/releases`) and check `SHA256SUMS`.
