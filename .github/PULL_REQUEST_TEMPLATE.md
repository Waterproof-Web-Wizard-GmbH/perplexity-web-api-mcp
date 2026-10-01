<!--
Do not include cookies, tokens, or account UUIDs in the diff, the description, or attached logs.
-->

## What changed

<!-- One or two sentences. Link the issue if there is one ("Fixes #123"). -->

## Why

<!-- The problem this solves. For changes that react to a Perplexity-side change, describe what Perplexity changed and how you found out. -->

## How it was tested

<!-- For example: unit tests, `cargo run --bin perplexity-web-api-mcp -- --check`, a tool call from an MCP client against the live API. Name the client and OS. -->

## Checklist

- [ ] `cargo fmt --all` leaves no changes
- [ ] `cargo clippy --workspace --all-targets --all-features -- -D warnings` passes
- [ ] `cargo build --workspace --all-targets` and `cargo test --workspace` pass
- [ ] README and examples are updated if behavior or configuration changed
- [ ] Commit messages follow Conventional Commits (`feat:`, `fix:`, `docs:`, `refactor:`, `perf:`)
- [ ] No cookies, tokens, or account UUIDs anywhere in this pull request

## Upstream

<!-- Does this fix apply to AntonIXO/perplexity-web-api-mcp as well? If yes, say so; it will be offered upstream. -->
