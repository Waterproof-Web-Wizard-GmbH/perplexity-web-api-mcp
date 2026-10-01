# Perplexity Web API MCP Server

MCP (Model Context Protocol) server that exposes Perplexity AI search, research, and reasoning as tools for Claude Code, Codex, Cursor, VS Code, Zed, and other MCP clients.

It talks to the same web endpoints the Perplexity website uses and authenticates with your own browser session. No Perplexity API key and no per-request fees; a free account works, a Pro subscription unlocks the Pro models and higher quotas.

## Why this fork

- [mishamyrt](https://github.com/mishamyrt/perplexity-web-api-mcp) wrote the original project. It was archived in April 2026 at release 0.11.0 after Perplexity changed the schema of its SSE responses and the parser stopped working.
- [AntonIXO](https://github.com/AntonIXO/perplexity-web-api-mcp) forked it, fixed the parsing of the new `blocks` SSE shape, and continued development up to v0.12.3.
- This repository starts from AntonIXO v0.12.3 and adds the fix for Perplexity's new session scheme (v0.13.0). Perplexity replaced NextAuth: the session cookie is now `__Secure-pplx.session.<account-uuid>`, and every request has to carry an `x-pplx-account` header. Without the header the backend does not fail; it quietly answers as an anonymous user, so searches return nothing and quotas read as exhausted. Details are in [What changed in v0.13.0](#what-changed-in-v0130).

Both earlier projects are MIT licensed, and so is this one. Fixes that also apply upstream will be offered to AntonIXO.

## Disclaimer

This is an unofficial project and is not affiliated with, endorsed by, or supported by Perplexity AI.

It uses Perplexity's undocumented web endpoints together with your own browser session. Those endpoints can change at any time without notice, and when they do, this server may stop working until someone fixes it. Using the web endpoints outside of a browser may conflict with Perplexity's Terms of Service. Use it at your own risk and review the terms that apply to your account.

Your session cookie grants full access to your Perplexity account. Treat it like a password: keep it out of shell history, commits, screenshots, and bug reports.

## Requirements

Prebuilt binaries are published for:

- macOS (arm64, x86_64)
- Linux (x86_64, aarch64)
- Windows (x86_64)

Other platforms can build from source (see [Building from source](#building-from-source)).

## Installation

### Installer script

macOS and Linux:

```bash
curl --proto '=https' --tlsv1.2 -LsSf https://github.com/dhuettner/perplexity-web-api-mcp/releases/latest/download/perplexity-web-api-mcp-installer.sh | sh
```

Windows (PowerShell):

```powershell
irm https://github.com/dhuettner/perplexity-web-api-mcp/releases/latest/download/perplexity-web-api-mcp-installer.ps1 | iex
```

The installer places the binary in `~/.cargo/bin/perplexity-web-api-mcp` (`%USERPROFILE%\.cargo\bin` on Windows). Make sure that directory is on your `PATH`, or use the absolute path in your MCP client configuration.

### Other options

- Platform archives and `.sha256` checksum files are on the [Releases page](https://github.com/dhuettner/perplexity-web-api-mcp/releases/latest).
- Build and install with Cargo (Rust 1.91 or newer):

  ```bash
  cargo install --git https://github.com/dhuettner/perplexity-web-api-mcp perplexity-web-api-mcp
  ```

  Windows builds need extra tooling for BoringSSL; see [CONTRIBUTING.md](CONTRIBUTING.md).

- Docker (local build, Streamable HTTP transport): see [Docker](#docker).

## Configuration

### Getting your session cookie

1. Log in to [perplexity.ai](https://www.perplexity.ai) in your browser.
2. Open Developer Tools (F12, or right-click and choose Inspect).
3. Go to Application, then Cookies, then `https://www.perplexity.ai`.
4. Find the cookie whose name starts with `__Secure-pplx.session.`. The part after the last dot is your account UUID.
5. Copy the full cookie **name** into `PERPLEXITY_SESSION_COOKIE_NAME` and the cookie **value** into `PERPLEXITY_SESSION_TOKEN`.

A JSON export from the Cookie-Editor browser extension contains the same two fields (`name` and `value`) if you prefer that route.

The account UUID for the `x-pplx-account` header is derived from the cookie name automatically. Session cookies expire; when the server starts answering with empty results, repeat these steps.

### Checking the session

The binary has a `--check` flag that performs the warm-up and reads your remaining Pro quota. It prints one line and exits:

```text
ok remaining_pro=<n>        # exit 0: the session is accepted
anonymous remaining_pro=0   # exit 1: Perplexity treats the session as anonymous (expired cookie, wrong name, missing header)
error <message>             # exit 1: request failed
```

Example for a launcher script:

```bash
export PERPLEXITY_SESSION_TOKEN="..."
export PERPLEXITY_SESSION_COOKIE_NAME="__Secure-pplx.session.7ca5..."
perplexity-web-api-mcp --check || { echo "Perplexity session is not valid" >&2; exit 1; }
exec perplexity-web-api-mcp
```

### Environment variables

Authentication:

- `PERPLEXITY_SESSION_TOKEN` (optional): Value of the session cookie. Required for `perplexity_research`, `perplexity_reason`, `perplexity_computer`, `perplexity_study`, `perplexity_document_review`, `perplexity_usage`, `perplexity_view_chat`, connectors, model selection, and file attachments. Omit it for [tokenless mode](#tokenless-mode).
- `PERPLEXITY_SESSION_COOKIE_NAME` (optional, recommended): Full name of the session cookie, for example `__Secure-pplx.session.7ca5...`. The account UUID for the `x-pplx-account` header is taken from the part after `__Secure-pplx.session.`. If unset, the legacy NextAuth cookie name (`next-auth.session-token`) is used and no account header is sent; this only works for accounts that are still on the old scheme.
- `PERPLEXITY_ACCOUNT_ID` (optional): Overrides the account UUID derived from the cookie name.
- `PERPLEXITY_NO_ACCOUNT_HEADER` (optional): Set to any value to disable the `x-pplx-account` header. The client then falls back to the legacy NextAuth flow, including the `/api/auth/csrf` request. Intended for debugging.
- `PERPLEXITY_DEBUG_COOKIES` (optional): Set to `1` to print the cookie jar, the account header, and the warm-up response to stderr. The output contains your session secrets; do not share it unredacted.

Models (all require a token):

- `PERPLEXITY_ASK_MODEL`: Model for `perplexity_ask`. Default `pro-auto`.
  Valid values: `turbo`, `pro-auto`, `pro-upgraded`, `sonar`, `nemotron-3-super`, `claude-4.6-sonnet`, `claude-4.6-opus`, `gemini-3.0-flash`, `gemini-3.0-pro`, `gpt-5-pro`, `gpt-5.3-codex`, `gpt-5.4`, `gpt-5.4-mini`, `gpt-5.2`, `gpt-5.2-pro`, `grok-4.1`.
- `PERPLEXITY_REASON_MODEL`: Model for `perplexity_reason`. Default `gemini-3.1-pro`.
  Valid values: `gemini-3.1-pro`, `gemini-3.0-flash-high`, `claude-4.6-sonnet-thinking`, `claude-4.6-opus-thinking`, `gpt-5-thinking`, `gpt-5.1-thinking`, `gpt-5.2-thinking`, `gpt-5.4-thinking`, `grok-4.1-reasoning`, `kimi-k2.5-thinking`.
- `PERPLEXITY_COMPUTER_MODEL`: Model for `perplexity_computer`. Default `claude-4.6-opus-thinking`.
  Valid values: `asi`, `asi-beta`, `claude-4.6-sonnet`, `claude-4.6-sonnet-thinking`, `claude-4.6-opus`, `claude-4.6-opus-thinking`, `gpt-5.4`, `kimi`, `qwen`.
- `raw:` escape hatch: each of the three model variables also accepts `raw:<preference>` to pass an arbitrary Perplexity preference string through unchanged, for models newer than this build's list (for example `PERPLEXITY_REASON_MODEL=raw:glm_5_2`). Each tool only accepts preferences from its own family: an "ask" model in `PERPLEXITY_ASK_MODEL`, a "reasoning" model in `PERPLEXITY_REASON_MODEL`. A model from the wrong family does not produce an error; Perplexity accepts the request and returns `answer: null`. If a tool starts returning `null` after a model change, check the family first (see [Discovering available models](#discovering-available-models)).

Timeouts and progress:

- `PERPLEXITY_TIMEOUT_SECS` (default `30`): Request timeout for fast modes (search, ask, reason, study).
- `PERPLEXITY_LONG_TIMEOUT_SECS` (default `600`): Request timeout for long-running modes (Deep Research, Computer, Document Review). Raise it if deep-research runs are cut off. This timeout lives inside this server's HTTP client only; your MCP client has its own tool-call timeout (see [Progress heartbeats](#progress-heartbeats-for-long-running-tools)).
- `PERPLEXITY_PROGRESS_INTERVAL_SECS` (default `10`): Interval for `notifications/progress` heartbeats during a tool call, sent only when the client's request included a `progressToken`.
- `PERPLEXITY_INCOGNITO` (default `true`): Whether requests use Perplexity's incognito mode (`true` or `false`).

### What changed in v0.13.0

- Cookie name: `__Secure-pplx.session.<account-uuid>` replaces `__Secure-next-auth.session-token`. Set it with `PERPLEXITY_SESSION_COOKIE_NAME`.
- Account header: every request carries `x-pplx-account: <account-uuid>`. Perplexity supports several signed-in accounts at once, one cookie each, and without the header the backend answers anonymously with HTTP 200 instead of returning an error.
- CSRF: `/api/auth/csrf` no longer returns JSON for accounts on the new scheme. The request is skipped whenever the account header is in use.
- `--check` flag and `PERPLEXITY_DEBUG_COOKIES` for diagnosis.

Full history is in [CHANGELOG.md](CHANGELOG.md).

### Discovering available models

The typed model lists above (`SearchModel`, `ReasonModel`, `ComputerModel` in `crates/perplexity-web-api/src/models.rs`) are a snapshot and go stale as Perplexity ships new models. Two live sources show what your account can use before you reach for `raw:`:

1. Model config endpoint (no login required): [`https://www.perplexity.ai/rest/models/config`](https://www.perplexity.ai/rest/models/config) returns every model the Perplexity frontend knows about, grouped by the mode it is valid for (`ask`, `reason`/copilot, `computer`/agentic, and so on), with its internal `preference` string. That string is the value you pass to `raw:<preference>`, and the grouping tells you which family a model belongs to.

   ```bash
   curl -s 'https://www.perplexity.ai/rest/models/config' | jq .
   ```

2. Rate limit endpoint (requires a logged-in session): [`https://www.perplexity.ai/rest/rate-limit/all`](https://www.perplexity.ai/rest/rate-limit/all) returns your remaining quotas per feature. The same data is exposed by the `perplexity_usage` tool and by `--check`.

If you find a model that is missing from the typed lists, use it right away via `raw:<preference>`, or open a pull request adding it to `models.rs` so it gets validation and shows up in the tool descriptions.

### Progress heartbeats for long-running tools

`perplexity_research`, `perplexity_computer`, and `perplexity_document_review` (and occasionally `perplexity_reason` on a heavy query) can hold Perplexity's SSE connection open for minutes without sending any bytes. From an MCP client's point of view this looks like a hung request.

The [MCP lifecycle spec](https://modelcontextprotocol.io/specification/2025-06-18/basic/lifecycle#timeouts) allows a client to reset its per-request timeout every time it receives a `notifications/progress` message for that request's `progressToken`. This is a client-side opt-in (`resetTimeoutOnProgress` in the TypeScript SDK, or the equivalent in other clients); a server cannot force it, and clients may still apply an absolute maximum.

This server implements the server side:

- If, and only if, the `tools/call` request includes `_meta.progressToken`, a background task sends `notifications/progress` every `PERPLEXITY_PROGRESS_INTERVAL_SECS` (default 10 s) for the duration of the call.
- `progress` is a monotonically increasing tick counter. Perplexity's streaming API exposes no percent-complete signal, and the spec only requires the value to increase. The `message` field says so in plain English for clients that show it.
- The heartbeat task is tied to the tool call and aborted when the call ends, whether by success, error, or cancellation.
- Without a `progressToken` no task is spawned.

`PERPLEXITY_LONG_TIMEOUT_SECS` still governs how long this server waits for Perplexity. The heartbeat governs how long your MCP client waits, provided the client supports timeout reset on progress and sends a `progressToken`. Many clients, including simple stdio wrappers, do not; in that case raise the client's own timeout setting.

## Client setup

All examples assume the binary is on your `PATH`. If it is not, replace `perplexity-web-api-mcp` with the absolute path, for example `/Users/you/.cargo/bin/perplexity-web-api-mcp`.

### Claude Code

```bash
claude mcp add perplexity \
  --env PERPLEXITY_SESSION_TOKEN="your-cookie-value" \
  --env PERPLEXITY_SESSION_COOKIE_NAME="__Secure-pplx.session.your-account-uuid" \
  -- perplexity-web-api-mcp
```

### Codex

```bash
codex mcp add perplexity \
  --env PERPLEXITY_SESSION_TOKEN="your-cookie-value" \
  --env PERPLEXITY_SESSION_COOKIE_NAME="__Secure-pplx.session.your-account-uuid" \
  -- perplexity-web-api-mcp
```

### Cursor, Claude Desktop, Windsurf

These clients share the `mcpServers` format:

| Client | Config file |
|--------|-------------|
| Cursor | `~/.cursor/mcp.json` |
| Claude Desktop | `claude_desktop_config.json` |
| Windsurf | `~/.codeium/windsurf/mcp_config.json` |

```json
{
  "mcpServers": {
    "perplexity": {
      "command": "perplexity-web-api-mcp",
      "env": {
        "PERPLEXITY_SESSION_TOKEN": "your-cookie-value",
        "PERPLEXITY_SESSION_COOKIE_NAME": "__Secure-pplx.session.your-account-uuid"
      }
    }
  }
}
```

### Zed

Add the following to `context_servers` in your [settings file](https://zed.dev/docs/configuring-zed.html#settings-files):

```json
{
  "context_servers": {
    "perplexity": {
      "command": "perplexity-web-api-mcp",
      "env": {
        "PERPLEXITY_SESSION_TOKEN": "your-cookie-value",
        "PERPLEXITY_SESSION_COOKIE_NAME": "__Secure-pplx.session.your-account-uuid"
      }
    }
  }
}
```

### VS Code

Add to `.vscode/mcp.json`:

```json
{
  "servers": {
    "perplexity": {
      "type": "stdio",
      "command": "perplexity-web-api-mcp",
      "env": {
        "PERPLEXITY_SESSION_TOKEN": "your-cookie-value",
        "PERPLEXITY_SESSION_COOKIE_NAME": "__Secure-pplx.session.your-account-uuid"
      }
    }
  }
}
```

### Other MCP clients

Most clients accept the `mcpServers` layout shown for Cursor. If yours does not, check its documentation for the stdio server format.

## Tokenless mode

The server runs without a session cookie when `PERPLEXITY_SESSION_TOKEN` is unset. In this mode:

- Only `perplexity_search` (links only) and `perplexity_ask` (answer with sources) are registered.
- Both use the `turbo` model. Setting `PERPLEXITY_ASK_MODEL`, `PERPLEXITY_REASON_MODEL`, or `PERPLEXITY_COMPUTER_MODEL` is an error.
- File attachments and connectors are unavailable.

## Docker

The image is built locally and serves the MCP over Streamable HTTP:

```bash
docker build -t perplexity-web-api-mcp .

docker run -d \
  -p 8080:8080 \
  -e PERPLEXITY_SESSION_TOKEN="your-cookie-value" \
  -e PERPLEXITY_SESSION_COOKIE_NAME="__Secure-pplx.session.your-account-uuid" \
  perplexity-web-api-mcp
```

The server listens at `http://localhost:8080/mcp`. The image is built with `--features streamable-http`; source builds need the same feature for HTTP transport.

Client configuration:

```json
{
  "mcpServers": {
    "perplexity": {
      "url": "http://localhost:8080/mcp"
    }
  }
}
```

Docker-specific variables:

| Variable | Default | Description |
|----------|---------|-------------|
| `MCP_TRANSPORT` | `streamable-http` | `stdio` or `streamable-http` (the latter requires the `streamable-http` cargo feature) |
| `MCP_HOST` | `0.0.0.0` | Address to bind |
| `MCP_PORT` | `8080` | Port to listen on |

The authentication, model, timeout, and incognito variables described in [Configuration](#configuration) apply unchanged.

## Available tools

### `perplexity_search`

Web search with the `turbo` model. Returns links, titles, and snippets only, no generated answer. Works without a token.

Parameters:

- `query` (required): The search query.
- `sources` (optional): Array of sources. Standard: `"web"`, `"scholar"`, `"social"`. Connectors (token and connected account required): `"google_drive"`, `"gcal"` / `"google_calendar"`, `"outlook"`, `"github_mcp_direct"` / `"github"`, `"hugging_face"` / `"huggingface"`, `"notion_mcp"` / `"notion"`, `"linear_alt"` / `"linear"`, `"slack_direct"` / `"slack"`, `"jira_mcp_merge"` / `"jira"`, `"confluence_mcp_merge"` / `"confluence"`, `"microsoft_teams_mcp_merge"` / `"teams"`, `"onedrive"`, `"sharepoint"`, `"dropbox"`, `"box"`. Unknown connector IDs are passed through unchanged. Defaults to `["web"]`.
- `language` (optional): Language code such as `"en-US"` (default).

File attachments are not supported by this tool.

### `perplexity_ask`

Answers a question with source citations. Uses `pro-auto` when authenticated (configurable via `PERPLEXITY_ASK_MODEL`), `turbo` in tokenless mode.

Parameters: same as `perplexity_search`, plus `files` (optional, requires token), see [File attachments](#file-attachments).

### `perplexity_reason`

Step-by-step reasoning with web grounding. Uses `gemini-3.1-pro` by default, configurable via `PERPLEXITY_REASON_MODEL`. Requires a token.

Parameters: same as `perplexity_ask`.

### `perplexity_research`

Deep, multi-source research using Perplexity's deep research mode (`pplx_alpha`). Slow (often more than 60 seconds). Requires a token.

Parameters: same as `perplexity_ask`.

### `perplexity_computer`

Runs a task with Perplexity Computer, the agentic mode that can browse, run code, and use connected services. Uses `claude-4.6-opus-thinking` by default, configurable via `PERPLEXITY_COMPUTER_MODEL`. Requires a token.

Parameters: same as `perplexity_ask`.

### `perplexity_study`

Tutor-style explanations in Perplexity's study mode. Requires a token.

Parameters: same as `perplexity_ask`.

### `perplexity_document_review`

Detailed analysis of uploaded documents in Perplexity's document review mode. Attach the documents via `files`. Requires a token.

Parameters: same as `perplexity_ask`.

### `perplexity_usage`

Returns your account's remaining quotas from `/rest/rate-limit/all`: Pro Search (weekly), Deep Research (monthly), Create Files & Apps, Browser Agent / Computer, plus per-source monthly limits. Requires a token. Useful when a request comes back empty and you want to rule out an exhausted quota.

The server also checks the relevant quota before each metered request and returns a clear error when it is exhausted, instead of the empty answer Perplexity would otherwise produce.

### `perplexity_view_chat`

Reads an existing Perplexity thread directly through the REST API and returns its content. Nothing is posted to the thread. Requires a token.

Parameters:

- `url_or_id` (required): Full `perplexity.ai/search/...` or `/computer/tasks/...` URL, thread UUID, or thread slug.
- `query` (optional): Accepted for compatibility; the thread content is returned as is.

## File attachments

`perplexity_ask`, `perplexity_reason`, `perplexity_research`, `perplexity_computer`, `perplexity_study`, and `perplexity_document_review` accept an optional `files` array. Requires a token.

Each entry needs:

- `filename` (required): Filename with extension, for example `"report.pdf"` or `"notes.txt"`.
- `text` (mutually exclusive with `data`): Plain-text content, for `.txt`, `.md`, `.csv`, `.json`, source code, and so on.
- `data` (mutually exclusive with `text`): Base64-encoded binary content, for `.pdf`, `.docx`, images, and so on.

Plain text:

```json
{
  "query": "Summarise the key points",
  "files": [
    {
      "filename": "notes.txt",
      "text": "Meeting notes: Q1 revenue up 12%..."
    }
  ]
}
```

Binary file:

```json
{
  "query": "What does this contract say about termination?",
  "files": [
    {
      "filename": "contract.pdf",
      "data": "JVBERi0xLjQK..."
    }
  ]
}
```

Several files can be passed in one request. They are uploaded to Perplexity's storage in parallel before the query is sent.

## Response format

`perplexity_search` returns web results only:

```json
{
  "web_results": [
    {
      "name": "Source name",
      "url": "https://example.com",
      "snippet": "Source snippet"
    }
  ]
}
```

The other query tools return the answer, the sources, and follow-up context:

```json
{
  "answer": "The generated answer text...",
  "web_results": [
    {
      "name": "Source name",
      "url": "https://example.com",
      "snippet": "Source snippet"
    }
  ],
  "follow_up": {
    "backend_uuid": "uuid-for-follow-up-queries",
    "attachments": []
  }
}
```

## Troubleshooting

- Empty results, `answer: null`, or quotas that read as `0` right after setup: Perplexity is treating the session as anonymous. Run `perplexity-web-api-mcp --check` with the same environment. Check that `PERPLEXITY_SESSION_COOKIE_NAME` is the full cookie name including the UUID and that the cookie has not expired.
- `Failed to fetch CSRF token`: the client is on the legacy NextAuth path. Set `PERPLEXITY_SESSION_COOKIE_NAME` so the account header is used and the CSRF request is skipped.
- `null` answers after changing a model variable: the model belongs to the other family. See [Discovering available models](#discovering-available-models).
- Deep research cut off: raise `PERPLEXITY_LONG_TIMEOUT_SECS` and the MCP client's own timeout. See [Progress heartbeats](#progress-heartbeats-for-long-running-tools).
- For anything else, set `PERPLEXITY_DEBUG_COOKIES=1` and read stderr. Redact the output before sharing it.

## Building from source

Build instructions, including optional cargo features and Windows prerequisites, are in [CONTRIBUTING.md](CONTRIBUTING.md).

## Contributing and security

Bug reports and pull requests are welcome; see [CONTRIBUTING.md](CONTRIBUTING.md). Please never paste session cookies or `PERPLEXITY_DEBUG_COOKIES` output into an issue.

Security issues go through the private reporting process described in [SECURITY.md](SECURITY.md), not through public issues.

## License

MIT. See [LICENSE](LICENSE). Copyright for the original code remains with the respective authors.

## Maintainer

Maintained by Dennis Hüttner, [Waterproof Web Wizard GmbH](https://waterproof-web-wizard.com/). Contributions are welcome; fixes that apply upstream will be offered to [AntonIXO](https://github.com/AntonIXO/perplexity-web-api-mcp).
