# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

Configuration-only repository that runs [autoupdate](https://github.com/rios0rios0/autoupdate) on a daily schedule via GitHub Actions. No source code to build — just YAML config and workflow files.

## Key Files

- `.autoupdate.yaml` — global autoupdate configuration: GPG key path and fork exclusion. It carries **no** `providers` block; the workflow appends a single-owner one per matrix job.
- `.github/workflows/autoupdate.yaml` — daily workflow (09:00 UTC), one matrix job per owner. Installs Flutter (which bundles the Dart SDK the runner image lacks), writes secrets to `.secure_files/` as files, renders a single-owner config, runs autoupdate, asserts the owner was reached, cleans up.
- `.github/workflows/claude.yaml` and `claude-code-review.yaml` — Claude Code assistant and PR review workflows (delegate to reusable workflows in `rios0rios0/.github`).
- `.github/workflows/release.yaml` — tags releases on push to `main` (delegates to reusable workflow in `rios0rios0/pipelines`).

## Architecture: File-Based Secrets

The workflow does **not** pass credentials via environment variables. Instead:
1. The matrix job's owner token (`secrets[matrix.owner.secret]`) and `GPG_PRIVATE_KEY` are written to `.secure_files/` as files.
2. The rendered config references these file paths (`token: '.secure_files/github_access_token.key'`, `gpg_key_path: '.secure_files/autoupdate.asc'`).
3. The cleanup step removes `.secure_files/` on every run (including failures).

## Architecture: One Job Per Owner

A GitHub fine-grained PAT is bound to a single resource owner, so one token can never span
`rios0rios0`, `medhub-life` and `prefy`. The workflow therefore fans out with
`strategy.matrix.owner`, each entry pairing an owner with the secret holding its token, and
`fail-fast: false` so one broken token does not cancel the others. The `Render Owner
Configuration` step appends a single-owner `providers` block to a copy of `.autoupdate.yaml`.

AutoUpdate logs a discovery failure and still exits `0`, so the `Assert Owner Was Reached` step
fails the job when the log contains `Failed to discover repos in`/`Failed to initialize provider`
or shows no `Run complete:` summary. A non-zero error count in that summary is downgraded to a
warning, not a failure.

## Required Secrets and Variables

Secrets: `GPG_PRIVATE_KEY`, plus one fine-grained PAT per owner — `PERSONAL_ACCESS_TOKEN`
(`rios0rios0`), `MEDHUB_ACCESS_TOKEN` (`medhub-life`), `PREFY_ACCESS_TOKEN` (`prefy`).
Every token's lifetime must be **366 days or less**; both organizations reject longer-lived
fine-grained tokens with a `403`.

Variables: `GIT_USER_NAME`, `GIT_USER_EMAIL`, `GIT_USER_SIGNINGKEY`.

## Commands

```bash
# Install autoupdate binary locally
curl -fsSL https://raw.githubusercontent.com/rios0rios0/autoupdate/main/install.sh | sh -s -- --install-dir . --force

# Verify it works
./autoupdate --help

# Dry run against a single owner (will fail without credentials — expected).
# `.autoupdate.yaml` has no providers block, so append one first, as the workflow does.
./autoupdate run --dry-run --config /path/to/rendered-config.yaml

# Validate YAML syntax (ignore missing document start warning)
yamllint .autoupdate.yaml
```

## Conventions

- `CHANGELOG.md` is generated and never edited by hand — a change writes its own fragment with
  `chlog new --kind <Kind> --body "..."`, and the kinds are the
  [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) categories.
- Commit messages and branch names follow conventions in the [Development Guide](https://github.com/rios0rios0/guide/wiki).
- Release branches use `bump/x.x.x` naming.

<!-- chlog:start -->
## Changelog (chlog) — MANDATORY

If the repository you are working in uses chlog (a `.chlog.yaml` or `.chlog.yml`
config file, or a `.changes/` directory, exists at the project root), the
following is binding and ALWAYS applies: whenever you make ANY change, you MUST
create a changelog fragment as part of the same change — automatically, without
being asked, before committing.

- Do NOT edit CHANGELOG.md directly; it is generated from fragments.
- Create the fragment with:
  `chlog new --kind <Kind> --body "<imperative description>"`
- Valid kinds: Added, Changed, Deprecated, Removed, Fixed, Security
- Choose the kind that best matches the change (e.g., new feature → Added,
  bug fix → Fixed, behavior change → Changed, removal → Removed, security fix → Security).
- If the change is backward-INCOMPATIBLE with the public API (a breaking
  change), you MUST add the `--breaking` flag:
  `chlog new --kind <Kind> --breaking --body "<description>"`.
  This is the ONLY thing that triggers a major version bump — the kind alone
  never does (per SemVer, major = incompatible change). When unsure whether a
  change breaks compatibility, ask the user instead of guessing.
- Fragments are YAML files in `.changes/unreleased/`; stage them with your commit.
- `chlog check` fails the build when a fragment is missing — never skip it.
<!-- chlog:end -->
