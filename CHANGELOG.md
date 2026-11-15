# Changelog

This is the Cardlove fork of [omouren/mcp-oauth-bundle](https://github.com/omouren/mcp-oauth-bundle),
maintained as `card-love/mcp-oauth-bundle` since 2026-11-12. Upstream's own releases end at `1.2`.
How the fork is kept: [docs/maintaining.md](docs/maintaining.md).

## 2.3.0 — 2026-11-15

- **Require `league/oauth2-server-bundle` `^2.0`** (was `^1.2`). Nothing in the bundle had to change:
  the tests and PHPStan pass on 2.0.0 for Symfony 7.4 and 8.1, its Doctrine mapping is byte-identical
  to 1.2.3, and the consuming backend ran its full suite green on it. League 2 itself needs Symfony
  7.4 or 8.1, so the `symfony/*` constraints say so now too (`^7.4|^8.1`).
- **Removed `LeagueCompatibilityPass` and `AccessTokenRepositoryDecorator`.** They emitted the RFC 8707
  claims for League 0.10, which had no `AccessTokenExtraClaimsResolveEvent`; every version we allow has
  it, so the pass returned at its first line.
- A consumer can drop `client.allow_plaintext_secrets` from its League config: 2.0 ignores it and
  reports a deprecation.

## 2.2.0 — 2026-11-15

- **Skip CIMD redirect URIs we cannot serve instead of refusing the client.** Native apps list
  private-use schemes next to their loopback URI (RFC 8252 §7.1): Raycast has seven app schemes and
  `http://127.0.0.1/callback`. 2.1.0 refused the whole document on the first such URI; now only the
  https and loopback-http URIs are stored, and a document left with none is still refused. League
  redirects only to stored URIs, so a skipped one cannot be used.

## 2.1.0 — 2026-11-12

The six pull requests left open upstream (all by Javier Eguiluz, 2026-10-06), merged unchanged:

- **Bind consent to the authorize request it was shown for**
  ([omouren#1](https://github.com/omouren/mcp-oauth-bundle/pull/1)). The consent page renders only
  what `ConsentListener` stored under a random `consent_id`, and "Allow" approves exactly that request,
  once. Before, the page took `client_name` from the query string, and an unused approval carried over
  to the next MCP authorize request in the session. `/oauth2/consent` without a valid id answers 400.
- **Require PKCE S256 for every MCP client** ([omouren#4](https://github.com/omouren/mcp-oauth-bundle/pull/4)).
  `plain` and a missing `code_challenge_method` are refused with `invalid_request`.
- **Harden the CIMD document fetch** ([omouren#7](https://github.com/omouren/mcp-oauth-bundle/pull/7)):
  `NoPrivateNetworkHttpClient`, no redirects, a 5 KB cap, a 10 s total limit, and `redirect_uris`
  restricted to https or loopback http.
- **Harden dynamic client registration** ([omouren#6](https://github.com/omouren/mcp-oauth-bundle/pull/6))
  and **accept `client_secret_basic` there** ([omouren#3](https://github.com/omouren/mcp-oauth-bundle/pull/3)).
  New optional settings `dcr.redirect_uri_schemes` and `dcr.rate_limiter`.
- **GitHub Actions CI** ([omouren#8](https://github.com/omouren/mcp-oauth-bundle/pull/8)), with the
  matrix moved to Symfony 7.4 and 8.1 — what league `^1.2` and mcp-bundle `^0.15` resolve to.

Not taken: [omouren#2](https://github.com/omouren/mcp-oauth-bundle/pull/2) (mcp-bundle 0.14, superseded
by 2.0.0) and, for now, [omouren#5](https://github.com/omouren/mcp-oauth-bundle/pull/5)
(`public_base_url`): its guard accepts an unresolved `%env()%` placeholder for `trusted_hosts`, so it
passes for every default Symfony app without protecting it.

## 2.0.0 — 2026-11-12

- Package renamed to `card-love/mcp-oauth-bundle`. The PHP namespace stays `Omouren\McpOauthBundle`,
  so consuming code does not change.
- Requires `symfony/mcp-bundle` `^0.15` (and with it `mcp/sdk` `^0.9`) and
  `league/oauth2-server-bundle` `^1.2`. Upstream required `^0.13` and `^0.10 || ^1.2`.
- No code change otherwise; identical to upstream `1.2` (`5dcf252`).
