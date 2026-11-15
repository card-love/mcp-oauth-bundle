# Maintaining this fork

`card-love/mcp-oauth-bundle` is maintained by Cardlove for its own backend (`card-love/cl-shop-backend`).
It is public, consumed as a Composer `vcs` repository, and not on Packagist.

## What stays fixed

- **The PHP namespace `Omouren\McpOauthBundle`.** Renaming it would touch every consumer for nothing.
- **Our tag line is 2.x.** Upstream's tags are `1.0`–`1.2`; a revived upstream `1.3` cannot clash.
- **No backward-compatibility ranges we do not run.** We require what the consumer actually uses
  (today mcp-bundle `^0.15`, league `^2.0`); widening is a deliberate step, not a default.

## Releasing

```bash
composer update
rm -rf var/cache          # see "Traps" — a stale test container hides wiring changes
vendor/bin/phpunit && vendor/bin/phpstan analyse
git push origin main      # CI runs the PHP × Symfony matrix
git tag 2.x.y && git push origin 2.x.y
```

Then, in the consumer: `composer update card-love/mcp-oauth-bundle`. Add the version to
[CHANGELOG.md](../CHANGELOG.md) in the same commit as the change.

## Upstream

`upstream` is a remote on omouren/mcp-oauth-bundle. `git fetch upstream` shows whether it moved; its
pull requests can be merged with `git fetch upstream pull/<n>/head:upstream-pr-<n>`. Record what was
taken, and what was not and why, in the changelog.

## Traps met so far

- **A stale test container.** The test kernel runs with `APP_DEBUG=0`, so it reuses `var/cache`
  without a freshness check. After a merge that changes service wiring, tests then see the old
  container — on 2026-11-12 that made the CIMD client look unprotected. Clear `var/cache` first.
- **Renaming the package unregisters the bundle in the consumer.** Symfony Flex removes the old
  package's bundle from `config/bundles.php`, and this package has no recipe to add it back. After
  switching a consumer from `omouren/…` to `card-love/…`, restore that line.
