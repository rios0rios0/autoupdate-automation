# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

When a new release is proposed:

1. Create a new branch `bump/x.x.x` (this isn't a long-lived branch!!!);
2. The Unreleased section on `CHANGELOG.md` gets a version number and date;
3. Open a Pull Request with the bump version changes targeting the `main` branch;
4. When the Pull Request is merged, a new Git tag must be created using <LINK TO THE PLATFORM TO OPEN THE PULL REQUEST>.

Releases to productive environments should run from a tagged version.
Exceptions are acceptable depending on the circumstances (critical bug fixes that can be cherry-picked, etc.).

## [Unreleased]

### Added

- added the `medhub-tech` and `prefy` organizations to the GitHub provider in `.autoupdate.yaml`, so repositories in both are discovered and dependency-updated alongside `rios0rios0`
- added a `Setup Flutter` step to the daily workflow so `dart` and `flutter` are on the `PATH` when AutoUpdate runs — the `ubuntu-latest` runner preinstalls Go, Node.js, Python, Ruby, Java and .NET but ships no Dart SDK, and AutoUpdate's Dart updater downgrades a missing toolchain to a warning, so Dart and Flutter repositories would have been skipped silently instead of failing loudly

### Security

- restricted the daily workflow's `GITHUB_TOKEN` to `contents: read`, the minimum `actions/checkout` needs — the job authenticates to GitHub with `PERSONAL_ACCESS_TOKEN` from `.secure_files/`, so it never needed the broader set of scopes it was inheriting from the repository default

## [0.2.1] - 2026-05-19

### Changed

- refreshed `CLAUDE.md` and `.github/copilot-instructions.md` to document the `release.yaml` workflow and update repository structure tree

## [0.2.0] - 2026-04-28

### Added

- created `CLAUDE.md` with repository guidance for Claude Code sessions

### Changed

- refreshed `.github/copilot-instructions.md` to match current `.autoupdate.yaml` config (file-based secrets, GPG signing, `exclude_forks`), corrected workflow schedule to 09:00 UTC, added missing secrets/variables documentation, and updated repository structure tree

## [0.1.0] - 2026-03-24

The changes weren't tracked until this version.
