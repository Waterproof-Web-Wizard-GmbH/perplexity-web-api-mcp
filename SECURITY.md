# Security Policy

Thank you for helping keep perplexity-web-api-mcp secure.

## Reporting a Vulnerability

Report security issues privately through GitHub's private vulnerability reporting for this repository: open the repository's **Security** tab and choose **Report a vulnerability**, or use this link:

https://github.com/dhuettner/perplexity-web-api-mcp/security/advisories/new

Please **do not** report security vulnerabilities through public issues, pull requests, or other public channels.

You should receive an acknowledgement within a few days. Fixes are published as a new release; the advisory is made public once a fixed version is available.

## What to Include

- A description of the vulnerability
- Steps to reproduce
- The potential impact
- A suggested fix, if you have one

## Scope

In scope: the Rust code in this repository, the release installers, and the documentation.

Out of scope: Perplexity's own services and endpoints. This project talks to them but does not control them. Please report issues in Perplexity's services to Perplexity.

## Handling Session Cookies

The session cookie configured in `PERPLEXITY_SESSION_TOKEN` grants full access to your Perplexity account. This project never writes it to disk, and it is only printed when you set `PERPLEXITY_DEBUG_COOKIES=1`.

- Never paste cookies, tokens, account UUIDs, or unredacted debug output into issues, pull requests, or advisories.
- If a cookie has leaked, log out of Perplexity in the browser it came from (or sign out of all sessions in your account settings) to invalidate it, then copy a fresh one.
