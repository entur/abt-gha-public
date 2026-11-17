# Changelog

All notable changes to this repository are documented here. The format is based on
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

Baseline: [v1.36](https://github.com/entur/abt-gha-public/releases/tag/v1.36) (2026-09-30).
Changes before v1.36 are not listed; see the git history and tags.

## [Unreleased]

### Added
- `cd`: when the Terraform plan has changes, the changelog Slack message links to the plan job, with the add/change/destroy counts.
- `safe-java-version` action: translates a Java version and a `setup-java` distribution into a safe version. A bare major (`17`, `21`, `25`) on Liberica becomes the patched build (`17.0.21+0.3.5`, `21.0.13+0.3.5`, `25.0.5+0.3.5`; CVE-2026-91403); full versions are passed through. Temurin has no patched build, so it is passed through with a warning, or fails with `fail-if-unpatched: true`. Covered by the `Test safe-java-version` workflow.

### Removed
- Reusable workflows with no known callers (#67): `gradle-release-sona`, `gradle-release-tag-sona`, `maven-release-sona`, `validate-jar-gradle-sona`, `validate-jar-maven-sona`, `notify-changelog`, `helm-deploy`.
- Reusable workflows whose only callers are archived, being archived, or a test repository (#67, #68): `maven-release`, `validate-jar-gradle`, `maven-open-source-increment-version-and-release-to-maven-central`, `gradle-open-source-increment-version-and-release-to-maven-central`, `maven-open-source-release-tag-to-maven-central`, `gradle-open-source-release-tag-to-maven-central`, `health-check`, `check-should-release`.
- The manual, on-tag and on-main release sections of the README, which documented the removed open-source release workflows (#68).

### Changed
- Runners: `ubuntu-latest` → `ubuntu-24.04` (#69).
- Same-repository action and workflow references use the `$/` self-repository syntax (requires runner 2.336.0 or newer), replacing branch and relative refs (#72).
- Reduced the number of Maven Central releases triggered by bot updates: `maybe-release-bot-updates-maven` and `-gradle` (#52).
- Default `java-version` is now the patched Liberica build `25.0.5+0.3.5` (BellSoft `25.0.5.0.3+5`, CVE-2026-91403) instead of plain `25`, which resolves to the unfixed `25.0.5+11`. Affects `build-jar-maven`, `maybe-release-bot-updates-maven`, `maybe-release-bot-updates-gradle` and the `partner-app-copilot-setup-steps-maven`/`-gradle` actions.
- `maven-open-source-verify`, `gradle-open-source-verify` and `build-jar-gradle`: default `java-version` changed from `21` to `25.0.5+0.3.5`. Callers that relied on the default now build with Java 25; pass `java-version` explicitly to keep an older version.
- The reusable workflows that call `actions/setup-java` (`build-jar-maven`, `build-jar-gradle`, `validate-jar-maven`, `maven-open-source-verify`, `gradle-open-source-verify`, `maybe-release-bot-updates-maven`/`-gradle`) resolve the version through `safe-java-version`. The default is a bare `25` again, and the patched build now lives only in the action. Callers passing a bare `17`, `21` or `25` on Liberica now get the patched build; explicit full versions are unchanged. Callers on another distribution get a warning.

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

### Fixed
- `cd`: the changelog Slack message (and approval link) was never posted when a job in the same run had already deployed to the environment, e.g. the descriptor upload. GitHub marks older deployments `inactive` once a newer one succeeds, and only deployments whose latest status was `success` were considered; now any deployment that ever reached `success` counts.

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
