# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

Configuration-only repository that runs [autoupdate](https://github.com/rios0rios0/autoupdate) on a daily schedule via GitHub Actions. No source code to build — just YAML config and workflow files.

## Key Files

- `.autoupdate.yaml` — autoupdate tool configuration: providers, GPG key path, fork exclusion.
- `.github/workflows/autoupdate.yaml` — daily workflow (09:00 UTC). Installs Flutter (which bundles the Dart SDK the runner image lacks), writes secrets to `.secure_files/` as files, runs autoupdate, cleans up.
- `.github/workflows/claude.yaml` and `claude-code-review.yaml` — Claude Code assistant and PR review workflows (delegate to reusable workflows in `rios0rios0/.github`).
- `.github/workflows/release.yaml` — tags releases on push to `main` (delegates to reusable workflow in `rios0rios0/pipelines`).

## Architecture: File-Based Secrets

The workflow does **not** pass credentials via environment variables. Instead:
1. `PERSONAL_ACCESS_TOKEN` and `GPG_PRIVATE_KEY` secrets are written to `.secure_files/` as files.
2. `.autoupdate.yaml` references these file paths (`token: '.secure_files/github_access_token.key'`, `gpg_key_path: '.secure_files/autoupdate.asc'`).
3. The cleanup step removes `.secure_files/` on every run (including failures).

## Required Secrets and Variables

Secrets: `PERSONAL_ACCESS_TOKEN`, `GPG_PRIVATE_KEY`.
Variables: `GIT_USER_NAME`, `GIT_USER_EMAIL`, `GIT_USER_SIGNINGKEY`.

## Commands

```bash
# Install autoupdate binary locally
curl -fsSL https://raw.githubusercontent.com/rios0rios0/autoupdate/main/install.sh | sh -s -- --install-dir . --force

# Verify it works
./autoupdate --help

# Dry run (will fail without credentials — expected)
./autoupdate run --dry-run

# Validate YAML syntax (ignore missing document start warning)
yamllint .autoupdate.yaml
```

## Conventions

- Follow [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) format in `CHANGELOG.md`.
- Commit messages and branch names follow conventions in the [Development Guide](https://github.com/rios0rios0/guide/wiki).
- Release branches use `bump/x.x.x` naming.
