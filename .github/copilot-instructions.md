# Autoupdate Automation Repository

This repository provides automated dependency and version management across multiple projects using the [autoupdate tool](https://github.com/rios0rios0/autoupdate). It is a configuration and automation repository containing a GitHub Actions workflow that runs the autoupdate tool daily to detect outdated dependencies and create Pull Requests to upgrade them.

**ALWAYS** reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

### Bootstrap and Test the Repository
- Install required system dependencies:
  - `sudo apt-get update && sudo apt-get install -y curl` -- takes 30-60 seconds
- Download and test autoupdate binary using the official install script:
  - `curl -fsSL https://raw.githubusercontent.com/rios0rios0/autoupdate/main/install.sh | sh -s -- --install-dir . --force` -- downloads and installs the binary
  - `./autoupdate --help` -- verify binary works

### Configuration Management
- The main configuration file is `.autoupdate.yaml` containing:
  - `gpg_key_path` pointing to the GPG signing key
  - `exclude_forks: true` to skip forked repositories
  - **No** provider definitions. A fine-grained PAT is bound to a single resource owner, so the
    workflow runs one matrix job per owner and appends a single-owner `providers` block (token
    reference plus organization list) to a copy of this file at runtime
- Token format supports inline values, `${ENV_VAR}` references, or file paths (current config uses a file path)
- Validate configuration syntax: `yamllint .autoupdate.yaml` -- takes <1 second
  - WARNING: yamllint will report missing document start "---" which is acceptable

### Testing Workflow Components
- Test autoupdate processing:
  - `./autoupdate run --dry-run` -- preview changes without creating PRs
  - With valid credentials: discovers repos and scans dependencies
  - Without credentials: fails with authentication errors (<1 second), which is expected

## Validation

### Manual Workflow Testing
- **ALWAYS** test workflow components after making configuration changes
- Run the complete workflow simulation:
  1. Download dependencies (30-60 seconds)
  2. Download autoupdate binary via install script (<5 seconds)
  3. Test binary execution (<1 second)
  4. Validate configuration syntax (<1 second)
- **NEVER CANCEL** workflow operations - all steps complete in under 2 minutes
- The actual GitHub Actions workflow runs daily at 09:00 UTC (6:00 AM BRT) and can be manually triggered

### Configuration Validation Steps
- Validate YAML syntax: `yamllint .autoupdate.yaml`
- Test autoupdate config parsing: `./autoupdate run --dry-run` (expect authentication errors without credentials)
- Verify provider organizations are accessible
- Check that token file paths or env var references correspond to configured repository secrets

## Common Tasks

### Repository Structure
```
.
├── .autoupdate.yaml         # Main autoupdate configuration (providers, GPG, exclusions)
├── .editorconfig            # Editor configuration
├── .github/
│   ├── copilot-instructions.md
│   ├── pull_request_template/
│   │   ├── bump.md          # PR template for dependency bump PRs
│   │   └── default.md       # Default PR template
│   ├── pull_request_template.md  # Legacy PR template
│   └── workflows/
│       ├── autoupdate.yaml       # Daily automation workflow
│       ├── claude-code-review.yaml  # Claude Code PR review workflow
│       ├── claude.yaml           # Claude Code assistant workflow
│       └── release.yaml          # Release tagging workflow
├── CHANGELOG.md             # Release history (Keep a Changelog format)
├── CLAUDE.md                # Claude Code assistant guidance
├── CONTRIBUTING.md          # Contribution guidelines
├── LICENSE                  # Project license
└── README.md                # Basic project description
```

### Key Configuration Files

#### .autoupdate.yaml
Holds only the settings shared by every owner:
```yaml
gpg_key_path: '.secure_files/autoupdate.asc'
exclude_forks: true
```

The `Render Owner Configuration` step copies it to `${RUNNER_TEMP}/autoupdate.yaml` and appends
the owner currently being processed, producing the file `./autoupdate run --config` reads:
```yaml
providers:
  - type: 'github'
    token: '.secure_files/github_access_token.key'
    organizations:
      - 'medhub-life'
```

#### .github/workflows/autoupdate.yaml
GitHub Actions workflow that:
- Runs daily at 09:00 UTC / 6:00 AM BRT (`cron: '0 9 * * *'`)
- Can be manually triggered via workflow_dispatch
- Fans out into one job per owner via `strategy.matrix.owner`, each entry pairing an owner name
  with the secret holding that owner's fine-grained PAT. `fail-fast: false` keeps one owner's
  broken token from cancelling the others
- Declares `permissions: contents: read` — the least privilege `actions/checkout` needs; GitHub API work uses the per-owner PAT, not `GITHUB_TOKEN`
- Installs Flutter via `subosito/flutter-action@v2` (stable channel, cache enabled) — the runner image has no Dart toolchain, and Flutter bundles the Dart SDK, so this puts both `flutter` and `dart` on the `PATH`
- Downloads latest autoupdate binary using the official install script
- Writes secrets to `.secure_files/` (GPG key and GitHub token as files)
- Validates the GPG key and the owner's token before running
- Renders a single-owner config, then runs `./autoupdate run --config` (credentials are read from the file paths in that config)
- Asserts the owner was actually reached: AutoUpdate logs discovery/provider failures and still
  exits 0, so the `Assert Owner Was Reached` step fails the job when the log contains
  `Failed to discover repos in` or `Failed to initialize provider`, or shows no `Run complete:`
  summary; a non-zero error count parsed from that summary becomes a warning, not a failure
- Cleans up `.secure_files/` on completion (always, even on failure)

### Workflow Secrets Required
The GitHub Actions workflow expects these repository secrets:
- `PERSONAL_ACCESS_TOKEN` - fine-grained PAT for the `rios0rios0` account
- `MEDHUB_ACCESS_TOKEN` - fine-grained PAT for the `medhub-life` organization
- `PREFY_ACCESS_TOKEN` - fine-grained PAT for the `prefy` organization
- `GPG_PRIVATE_KEY` - Armored GPG private key for commit signing (written to `.secure_files/autoupdate.asc`)

The matrix job writes its own owner's token to `.secure_files/github_access_token.key`. A
fine-grained PAT is bound to a single resource owner, so one token cannot cover all three, and
every token's lifetime must be **366 days or less**: both organizations reject longer-lived
fine-grained tokens with `403 ... forbids access via a fine-grained personal access tokens if
the token's lifetime is greater than 366 days`. To add an owner, add a matrix entry and its
secret — no other file changes.

And these repository variables:
- `GIT_USER_NAME` - Git committer name
- `GIT_USER_EMAIL` - Git committer email
- `GIT_USER_SIGNINGKEY` - GPG signing key ID

### Expected Timing
- **apt-get update && install**: 30-60 seconds
- **autoupdate install script**: <5 seconds
- **autoupdate execution**: <1 second
- **configuration validation**: <1 second
- **Complete workflow test**: <2 minutes total

### Common Failure Scenarios
- **"at least one provider must be configured"**: Check `.autoupdate.yaml` has valid provider entries
- **"providers[0].token is required"**: Ensure token file path or env var is configured
- **"providers[0].organizations must have at least one entry"**: Add at least one organization
- **"ERROR: GPG key file is empty"**: `GPG_PRIVATE_KEY` secret is not set or is empty
- **"ERROR: GPG key file does not contain valid armored PGP data"**: Secret must contain `gpg --export-secret-key --armor` output
- **"ERROR: the '<SECRET>' secret is empty or not set"**: the matrix owner's PAT secret is missing
- **`403 ... forbids access via a fine-grained personal access tokens`**: the owner's token was minted with a lifetime over 366 days; re-mint it with 366 days or less
- **Missing secrets**: Workflow will fail if the owner's PAT or `GPG_PRIVATE_KEY` secrets are not configured

### Making Changes
- **ALWAYS** validate configuration changes with `yamllint .autoupdate.yaml`
- **ALWAYS** test autoupdate can parse a rendered config with `./autoupdate run --dry-run --config <file>`
- **ALWAYS** verify provider organizations are valid and accessible
- No build process required - this is a pure configuration repository

### Integration Points
- Main autoupdate tool repository: https://github.com/rios0rios0/autoupdate
- Managed repositories discovered via GitHub API from configured organizations
- GitHub Actions for automation scheduling

### Debugging
- Check workflow runs in GitHub Actions tab
- Review autoupdate logs for authentication and processing errors
- Use `--dry-run` flag to preview changes without creating PRs
- Use `--verbose` flag for detailed logging
- Ensure repository secrets are properly configured
- Test configuration changes in a fork before applying to main repository
