# Changelog

All notable changes to this repository are documented here. The format is based on
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

Baseline: [v1.36](https://github.com/entur/abt-gha-public/releases/tag/v1.36) (2026-09-30).
Changes before v1.36 are not listed; see the git history and tags.

## [Unreleased]

### Removed
- Reusable workflows with no known callers (#67): `gradle-release-sona`, `gradle-release-tag-sona`, `maven-release-sona`, `validate-jar-gradle-sona`, `validate-jar-maven-sona`, `notify-changelog`, `helm-deploy`.
- Reusable workflows that had a single consumer each (#67, #68): `maven-release`, `validate-jar-gradle`, `maven-open-source-increment-version-and-release-to-maven-central`, `gradle-open-source-increment-version-and-release-to-maven-central`, `maven-open-source-release-tag-to-maven-central`, `gradle-open-source-release-tag-to-maven-central`, `health-check`, `check-should-release`.
- The manual, on-tag and on-main release sections of the README, which documented the removed open-source release workflows (#68).

### Changed
- Runners: `ubuntu-latest` → `ubuntu-24.04` (#69).
- Same-repository action and workflow references use the `$/` self-repository syntax (requires runner 2.336.0 or newer), replacing branch and relative refs (#72).
- Reduced the number of Maven Central releases triggered by bot updates: `maybe-release-bot-updates-maven` and `-gradle` (#52).

### Dependency updates (#70)

| Dependency | From | To |
|---|---|---|
| `actions/cache` (incl. `restore`, `save`) | v5.0.5 | v6.1.0 |
| `actions/checkout` | v7.0.0 | v7.0.1 |
| `actions/setup-java` | v5.3.0 | v6.0.1 |
| `entur/gha-slack` | v2 | v3 |
| `entur/gha-terraform` | v2 | v3 |
| `entur/gha-helm` | v1 | v2 |

`gradle/actions/setup-gradle` stays on v5.0.2 (v6 has licensing issues).

### Security

| Area | Change | PR |
|---|---|---|
| Script injection | `inputs.*` passed via `env:` instead of interpolated into `run:` in `cd` (approve), `gradle-open-source-verify` (`log-level`), `maybe-release-bot-updates-maven`/`-gradle`, `upload-file-to-bucket` and `validate-jar-maven` | #71 |
| Token permissions | `permissions:` added to jobs that had none (`contents: read`, or `{}` for summary-only jobs) | #71 |
| Credentials | `persist-credentials: false` on all `actions/checkout` steps in workflows | #71 |
| Supply chain | `lint-proto` uses the SHA-pinned `bufbuild/buf-setup-action` instead of an unverified `curl` download | #71 |
| Supply chain | Action refs to a mutable branch replaced by `$/` self-repository references | #72 |
| Secrets | `secrets: inherit` removed from the `gha-terraform` plan and apply calls, which use no secrets | #73 |

[Unreleased]: https://github.com/entur/abt-gha-public/compare/v1.36...main
