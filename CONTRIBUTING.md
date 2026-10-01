# Contributing Guide

Thank you for your interest in contributing to the Perplexity Web API MCP Server. This document covers how to build the project, what checks to run, and how to submit changes.

This repository is maintained by Dennis Hüttner (Waterproof Web Wizard GmbH). It descends from the fork by [AntonIXO](https://github.com/AntonIXO/perplexity-web-api-mcp) and the original project by [mishamyrt](https://github.com/mishamyrt/perplexity-web-api-mcp). Fixes that also apply upstream will be offered to AntonIXO; if you want your change to go upstream too, say so in the pull request.

## Requirements

- **Rust 1.91+** (edition 2024)
- **Cargo** (comes with Rust)
- A Perplexity AI account (for manual testing against the live API)

## Getting Started

### 1. Fork and Clone

```bash
git clone https://github.com/YOUR_USERNAME/perplexity-web-api-mcp.git
cd perplexity-web-api-mcp
```

### 2. Build the Project

#### macOS / Linux

```bash
cargo build --workspace --all-targets
```

To build the optimized MCP binary directly:

- `cargo build --profile dist --bin perplexity-web-api-mcp` builds the smaller `stdio`-only binary.
- `cargo build --profile dist --bin perplexity-web-api-mcp --features streamable-http` enables the optional Streamable HTTP transport.

#### Windows

Building on Windows requires additional tools because this project depends on BoringSSL (via the `rquest` crate).

**Prerequisites:**

- [Rust](https://rustup.rs)
- [Visual Studio Build Tools](https://visualstudio.microsoft.com/visual-cpp-build-tools/) with the **C++ desktop development** workload
- CMake, NASM, Ninja, and LLVM (libclang)

**Using the build script:**

The included `build.ps1` script downloads and sets up the required build tools into a local `.build-tools` directory:

```powershell
# First build (downloads dependencies automatically)
powershell -ExecutionPolicy Bypass -File build.ps1

# Skip dependency download (if tools are already installed system-wide)
powershell -ExecutionPolicy Bypass -File build.ps1 -SkipDeps

# Clean build (removes .build-tools and target directories)
powershell -ExecutionPolicy Bypass -File build.ps1 -Clean
```

> **Note:** The build must use `--release` mode. Debug builds cause a CRT mismatch between BoringSSL (`/MDd`) and Rust (`/MD`), resulting in linker errors.

### 3. Run Tests

```bash
cargo test --workspace
```

### 4. Check a Live Session

Set the two authentication variables and run the health probe:

```bash
export PERPLEXITY_SESSION_TOKEN="your-cookie-value"
export PERPLEXITY_SESSION_COOKIE_NAME="__Secure-pplx.session.your-account-uuid"
cargo run --bin perplexity-web-api-mcp -- --check
```

`ok remaining_pro=<n>` means the session is accepted. See the README for what the other outputs mean.

## Project Structure

```
perplexity-web-api-mcp/
├── crates/
│   ├── perplexity-web-api/       # Core API client library
│   │   ├── src/
│   │   │   ├── auth.rs           # Session cookie handling
│   │   │   ├── client.rs         # HTTP client, account header, request handling
│   │   │   ├── config.rs         # API configuration constants
│   │   │   ├── error.rs          # Error types
│   │   │   ├── models.rs         # Typed model lists
│   │   │   ├── parse.rs          # Response parsing
│   │   │   ├── rate_limit.rs     # Quota / rate-limit types
│   │   │   ├── sse.rs            # Server-Sent Events stream handling
│   │   │   ├── types.rs          # Request/response types
│   │   │   └── upload.rs         # File upload functionality
│   │   ├── examples/             # Usage examples
│   │   └── tests/                # Integration tests
│   └── perplexity-web-api-mcp/   # MCP server binary
│       └── src/
│           ├── main.rs           # Entry point, env parsing, --check
│           └── server.rs         # MCP tool implementations
├── docs/                         # GitHub Pages landing page
├── Cargo.toml                    # Workspace configuration
└── AGENTS.md                     # AI agent guidelines
```

## Development Guidelines

### Workspace Dependencies

- Add new dependencies to the **workspace root** `Cargo.toml` under `[workspace.dependencies]`
- Reference them in crate `Cargo.toml` files via workspace resolution:

```toml
[dependencies]
new-dep = { workspace = true }
# or with features:
new-dep = { workspace = true, features = ["feature-name"] }
```

### Async Rust Best Practices

This project uses **Tokio** for the async runtime. Follow these guidelines:

- **Never block the runtime**: avoid `std::thread::sleep`, blocking IO, or heavy CPU work on async tasks
  - Use `tokio::task::spawn_blocking` if blocking is unavoidable
- **Propagate cancellation**: respect `.await` cancellation points
- **Add timeouts**: use `tokio::time::timeout` for external IO operations
- **Limit concurrency**: prefer bounded channels/streams; avoid unbounded buffering
- **Minimize cloning**: prefer explicit lifetimes/ownership; use `Arc` intentionally
- **No panics in library code**: avoid `unwrap()`/`expect()` in runtime paths; return typed errors

### Error Handling

- Use structured errors with `thiserror`
- Include actionable context in error messages
- Preserve error sources with `#[source]` for debugging
- Never leak secrets (session cookies, tokens, account IDs) in error messages or logs. Debug output that has to contain them belongs behind `PERPLEXITY_DEBUG_COOKIES` and goes to stderr.

### Code Style

- Match existing patterns in each module
- Prefer small, testable functions
- Document public APIs with doc comments
- Update the README and examples when behavior or configuration changes

## Before Submitting

### Required Checks

Always run these commands before committing:

```bash
# Format code
cargo fmt --all

# Lint with clippy (must pass with no warnings)
cargo clippy --workspace --all-targets --all-features -- -D warnings

# Build all targets
cargo build --workspace --all-targets

# Run tests
cargo test --workspace
```

### Commit Messages

The changelog is generated with git-cliff from [Conventional Commits](https://www.conventionalcommits.org/), so the prefix decides where a change shows up:

- `feat:` new functionality
- `fix:` bug fixes
- `docs:` documentation only
- `refactor:` code changes without behavior change
- `perf:` performance improvements
- `chore:`, `test:`, `ci:` are not listed in the changelog

Keep the first line under 72 characters and reference issues when applicable ("Fixes #123").

Examples:
```
feat: add timeout configuration to ClientBuilder
fix: parse SSE stream for malformed events
docs: describe the --check flag
```

## Pull Request Process

1. **Create a branch** from the default branch:
   ```bash
   git checkout -b fix/your-change
   ```

2. **Make your changes** following the guidelines above

3. **Run all checks** listed under Required Checks

4. **Push and open a pull request**. The template asks for:
   - What changed and why
   - How you tested it (including `--check` or a tool call against the live API, if relevant)
   - Confirmation that no cookies, tokens, or account IDs are in the diff or the description

5. **Address review feedback**

## Testing

### Running Examples

Examples that talk to the live API need a session:

```bash
export PERPLEXITY_SESSION_TOKEN="your-cookie-value"
export PERPLEXITY_SESSION_COOKIE_NAME="__Secure-pplx.session.your-account-uuid"
```

Then run, for example:

```bash
cargo run -p perplexity-web-api --example basic
cargo run -p perplexity-web-api --example streaming
cargo run -p perplexity-web-api --example with_cookies
cargo run -p perplexity-web-api --example with_file
```

### Adding Tests

- Add unit tests in the same file as the code being tested
- Add integration tests in `crates/perplexity-web-api/tests/` if needed
- Ensure tests are deterministic and do not require external services

## Reporting Bugs

Use the bug report template in this repository's issue tracker. It asks for the version, operating system, MCP client, and the output of `--check`. Never paste cookies, tokens, or unredacted `PERPLEXITY_DEBUG_COOKIES` output into an issue.

Security problems go through private vulnerability reporting, not the public tracker; see [SECURITY.md](SECURITY.md).

## License

By contributing, you agree that your contributions will be licensed under the MIT License.
