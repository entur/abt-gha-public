# Changelog

All notable changes to this repository are documented here. The format is based on
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

Baseline: [v1.36](https://github.com/entur/abt-gha-public/releases/tag/v1.36) (2026-09-30).
Changes before v1.36 are not listed; see the git history and tags.

## [Unreleased]

### Removed
- Reusable workflows with no known callers: `gradle-release-sona`, `gradle-release-tag-sona`, `maven-release-sona`, `validate-jar-gradle-sona`, `validate-jar-maven-sona`, `notify-changelog`, `helm-deploy` (#67).
- `maven-release` and `validate-jar-gradle`, which were only used by `entur/abt-parent` (#67).
- `maven-open-source-increment-version-and-release-to-maven-central`, `gradle-open-source-increment-version-and-release-to-maven-central`, `maven-open-source-release-tag-to-maven-central`, `gradle-open-source-release-tag-to-maven-central`, `health-check` and `check-should-release`, which were only used by `abt-gha-testing` and `abt-offline-control-deploy`. The matching release sections were removed from the README (#68).

### Changed
- Runners: `ubuntu-latest` → `ubuntu-24.04` (#69).
- Actions: `actions/cache` v6.1.0, `actions/checkout` v7.0.1, `actions/setup-java` v6.0.1, `entur/gha-slack` v3, `entur/gha-terraform` v3, `entur/gha-helm` v2. `gradle/actions/setup-gradle` stays on v5.0.2 (#70).
- Same-repository action and workflow references use the `$/` self-repository syntax (requires runner 2.336.0 or newer); this replaces the `@reduceMavenReleases` branch refs (#72).
- The reduced Maven Central release flow: `maybe-release-bot-updates-maven` and `-gradle` (#52).

### Security
- Workflow inputs are passed via `env:` instead of being interpolated into `run:` scripts; `permissions:` added to jobs that had none; `persist-credentials: false` on checkouts; `lint-proto` uses the pinned `bufbuild/buf-setup-action` instead of an unverified download (#71).
- Dropped `secrets: inherit` from the `gha-terraform` plan/apply calls (#73).

[Unreleased]: https://github.com/entur/abt-gha-public/compare/v1.36...main
