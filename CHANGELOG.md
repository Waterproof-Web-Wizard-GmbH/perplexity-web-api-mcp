# Changelog

All notable changes to this project will be documented in this file.

## [0.13.0](https://github.com/dhuettner/perplexity-web-api-mcp/compare/v0.12.3..v0.13.0) - 2026-10-01

First release from [dhuettner/perplexity-web-api-mcp](https://github.com/dhuettner/perplexity-web-api-mcp), based on AntonIXO v0.12.3.

### 🐛 Bug Fixes

- Authenticate against Perplexity's per-account session scheme: the session cookie is now `__Secure-pplx.session.<account-uuid>` and every request carries the `x-pplx-account` header. Without it Perplexity silently answers as an anonymous user.
- Skip the NextAuth `/api/auth/csrf` request when the account header is in use; the endpoint no longer returns JSON.
- `perplexity_view_chat` sends the account header too, so private threads are read as the logged-in user; its tool description no longer claims a follow-up turn is added.

### 🚀 Features

- `PERPLEXITY_SESSION_COOKIE_NAME` sets the session cookie name; the account id for the header is derived from it.
- `PERPLEXITY_NO_ACCOUNT_HEADER` disables the account header, `PERPLEXITY_DEBUG_COOKIES=1` prints the cookie jar and warm-up response for diagnosis.
- `--check` flag: prints the remaining Pro quota and exits non-zero when the session is anonymous or invalid.

### ⚙️ Miscellaneous Tasks

- Release binaries and shell/PowerShell installers are published from this repository; the npm package is no longer built.

## [0.12.3](https://github.com/AntonIXO/perplexity-web-api-mcp/compare/v0.12.2..v0.12.3) - 2026-08-17

### 🚀 Features

- Add direct REST API thread reading (`read_thread`) without synthetic AI search queries
- Add UUID extraction support for `/computer/tasks/<UUID>` URLs in `extract_thread_uuid_or_slug`

### 🚜 Refactor

- Completely remove synthetic follow-up query system from `perplexity_view_chat` in favor of direct API thread reading

## [0.12.2](https://github.com/AntonIXO/perplexity-web-api-mcp/compare/v0.12.1..v0.12.2) - 2026-08-03

### 🐛 Bug Fixes

- Parse Perplexity SSE `blocks` response shape (`markdown_block` + `web_result_block`); keep legacy `text`/FINAL path first so older payloads still work ([#2](https://github.com/AntonIXO/perplexity-web-api-mcp/pull/2))

### ⚙️ Miscellaneous Tasks

- Bump workspace deps: rmcp 1.8, tokio 1.53, serde 1.0.229, serde_json 1.0.151, uuid/bytes/memchr/futures/thiserror/schemars
- Note: `rquest` remains pinned via Cargo.lock — every crates.io version is yanked after rename to `wreq` (migration deferred)

## [0.11.0](https://github.com/mishamyrt/perplexity-web-api-mcp/compare/v0.10.0..v0.11.0) - 2026-04-08

### 🚀 Features

- Emulate chrome 136
- Add incognito disable

### 🚜 Refactor

- Split streaming, remove some features to decrease size

## [0.10.0](https://github.com/mishamyrt/perplexity-web-api-mcp/compare/v0.9.0..v0.10.0) - 2026-03-29

### 🚀 Features

- Add batch file upload api
- Add file upload support

## [0.9.0](https://github.com/mishamyrt/perplexity-web-api-mcp/compare/v0.8.0..v0.9.0) - 2026-03-17

### 🚀 Features

- Add streamable http transport
- Add docker image

### 🐛 Bug Fixes

- Correct example model name
- Avoid panic on base url parsing

### 🚜 Refactor

- Use tracing for client error logging
- Remove unreachable auth→hashmap convertion
- Remove unrechable tokenless checks
- Simplify FollowUpContext default constructor
- Avoid error variants discard on upload
- Avoid wildcard on mode matching
- Simplify sse parsing

### ⚡ Performance

- Simplify file upload serializing
- Avoid sse values re-serialization

## [0.8.0](https://github.com/mishamyrt/perplexity-web-api-mcp/compare/v0.7.0..v0.8.0) - 2026-03-15

### 🚀 Features

- Split search/ask tools, adapt official tool instructions
- Add annotations

### 🐛 Bug Fixes

- Provide correct mcp name and version

## [0.7.0](https://github.com/mishamyrt/perplexity-web-api-mcp/compare/v0.6.1..v0.7.0) - 2026-03-14

### 🚀 Features

- Use sonar for authenticated users by default
- Sync model list

### 📚 Documentation

- Add security guide
- Add code of conduct

## [0.6.1](https://github.com/mishamyrt/perplexity-web-api-mcp/compare/v0.5.0..v0.6.1) - 2026-03-14

### 🚀 Features

- Add tokenless (unauthorized) mode support

### 📚 Documentation

- Add Zed setup guide

## [0.5.0](https://github.com/mishamyrt/perplexity-web-api-mcp/compare/v0.4.0..v0.5.0) - 2026-03-02

### 🚀 Features

- Add correct single handler
- Add search model selection
- Add reason model selection

### 🐛 Bug Fixes

- Use correct model names

### 🚜 Refactor

- Improve error handling, align constants
- Avoid parsing outside api
- Use single delimiter finder
- Move model names to constants
- Commonize auth usage

### 📚 Documentation

- Align os/arch support
- Fix npm link

## [0.4.0](https://github.com/mishamyrt/perplexity-web-api-mcp/compare/v0.3.1..v0.4.0) - 2026-02-17

### 🚀 Features

- Add Windows x86 support

### 📚 Documentation

- Update cookie name

## [0.3.1](https://github.com/mishamyrt/perplexity-web-api-mcp/compare/v0.3.0..v0.3.1) - 2026-01-23

### 📚 Documentation

- Add web_results example
- Add more docs

## [0.3.0](https://github.com/mishamyrt/perplexity-web-api-mcp/compare/v0.2.1..v0.3.0) - 2026-01-23

### 🚀 Features

- Add links to results

### 📚 Documentation

- Add contributing guide

## [0.2.1](https://github.com/mishamyrt/perplexity-web-api-mcp/compare/v0.2.0..v0.2.1) - 2026-01-08

### 🚀 Features

- Send request as incognito

### 🚜 Refactor

- Split app and server
- Improve typing

### 📚 Documentation

- Describe configuration
- Add license
- Add agents md

## [0.2.0](https://github.com/mishamyrt/perplexity-web-api-mcp/compare/v0.1.1..v0.2.0) - 2026-01-08

### 🚀 Features

- Add prefix to envs

## [0.1.0](https://github.com/mishamyrt/perplexity-web-api-mcp/compare/..v0.1.0) - 2026-01-07

### 🚀 Features

- Add mcp

<!-- generated by git-cliff -->
