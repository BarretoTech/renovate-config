# @BarretoTech/renovate-config

Shared [Renovate](https://docs.renovatebot.com/) configuration for every repository in the Barreto Tech enterprise: the `archgate`, `jack-for-work` and `BarretoTech` organizations. It is the enterprise's only Renovate preset; `archgate/renovate-config` extends it unchanged, so repositories that still name that preset get the same rules.

## Usage

Each repository should have a `renovate.json` containing:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["local>BarretoTech/renovate-config"]
}
```

Repo-specific overrides can be added alongside the `extends` array.

## What's included

### Base configuration

Extends [`config:best-practices`](https://docs.renovatebot.com/presets-config/#configbest-practices), which includes:

- `config:recommended` (dependency dashboard, semantic commits, monorepo grouping)
- Pin devDependencies
- Pin Docker image digests
- Pin GitHub Action digests
- npm minimum release age (overridden to **7 days** — see below)
- Weekly lock file maintenance
- Config migration PRs

### Security hardening

- **7-day minimum release age** for all updates (overrides the 3-day `config:best-practices` default)
- **OpenSSF Scorecard** badges on PRs for supply-chain visibility
- **OSV vulnerability alerts** enabled
- **Vulnerability PRs** skip both the release-age delay **and** the monthly schedule (`schedule: ["at any time"]`), and get priority `10`
- **GitHub Actions pinned to SHA digests**
- **Major updates require dashboard approval** before PRs are created

### Noise reduction

- **Non-major devDependencies grouped** into a single PR per repo
- **Non-major production dependencies grouped** into a single PR per repo
- **Node.js, npm, and GitHub CLI minor and patch updates grouped** into a single PR
- **Bun, Archgate CLI, Moon, and Proto pins grouped** by tool so the pins for each tool update together; the Bun group also includes `@types/bun`
- **Docs dependencies grouped** into a single PR, committed as `chore(deps)` so a docs-site bump never triggers a release
- **GitHub Actions grouped** into a single PR
- **Java/Maven dependencies grouped** into a single PR (all update types)
- **.NET/NuGet dependencies grouped** into a single PR (all update types)
- **Python dependencies grouped** into a single PR (all update types)
- **Go module dependencies grouped** into a single PR (all update types)
- **Ruby/Bundler dependencies grouped** into a single PR (all update types)
- **Linters, formatters, and type packages auto-merged** on minor/patch
- **Schedule:** PRs created **only on the last Sunday of the month** (Sunday between the 25th and 31st, before 6am Europe/Oslo)
- **Automerges** are batched into the same monthly window
- **Rate limited:** max 20 concurrent PRs, no per-hour cap (so the full monthly batch can land in one run)

### Custom managers

| File | Pins tracked by the preset's regex managers |
| --- | --- |
| `.prototools` | Zizmor, Stripe CLI, and Archgate CLI versions; tagged `github://archgate/proto-plugin@v…` references |
| `.archgate/version` | Archgate CLI version |
| `.moon/workspace.yml` | Moon version in `versionConstraint` |
| `.moon/toolchains.yml` | Proto version in `proto.version` |
| `.github/workflows/*.yml` and `*.yaml` | `proto-version` passed to `moonrepo/setup-toolchain` or `BarretoTech/setup-toolchain` |

Renovate's built-in [Proto manager](https://docs.renovatebot.com/modules/manager/proto/) handles supported tools in `.prototools`, including Bun, GitHub CLI, Node.js, npm, Moon, and Proto. The built-in npm manager handles `engines.bun` in `package.json`. The preset does not extract those pins a second time. Untagged Archgate plugin references and version aliases such as `latest` are not updated by the custom managers.

### Ignored paths

- **Archgate rule fixtures** under `.archgate/fixtures/` are ignored, at any depth. They are written to pass or to violate a rule, so Renovate neither lists nor updates them.
- The `config:best-practices` ignore list (`node_modules`, `bower_components`, `vendor`, `examples`, `__tests__`, `test`, `tests`, `__fixtures__`) is repeated in `default.json`, because Renovate replaces `ignorePaths` and does not merge it. NuGet keeps its shorter list, which leaves test projects updatable.

A repository that sets its own `ignorePaths` replaces this whole list and must repeat the entries it still wants, `**/.archgate/fixtures/**` included.

## Overriding for a specific repo

Add `packageRules` or `ignorePresets` in the repo's own `renovate.json`:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["local>BarretoTech/renovate-config"],
  "packageRules": [
    {
      "description": "Repo-specific rule",
      "matchPackageNames": ["some-package"],
      "automerge": false
    }
  ]
}
```

## Presets reference

| Preset | Purpose |
| --- | --- |
| `config:best-practices` | Recommended + security defaults |
| `security:openssf-scorecard` | Supply-chain scorecard badges |
| `:approveMajorUpdates` | Dashboard gate for major bumps |
| `:enableVulnerabilityAlerts` | Vuln alert PRs |
| `:automergeRequireAllStatusChecks` | All CI must pass before automerge |
| `:rebaseStalePrs` | Keep PRs up to date with base |
| `:timezone(Europe/Oslo)` | Schedule timezone |
| `:gitSignOff` | Add DCO sign-off line to commits |

To opt out of the monthly schedule for a specific repo, override `schedule` and `automergeSchedule` in the repo config.
