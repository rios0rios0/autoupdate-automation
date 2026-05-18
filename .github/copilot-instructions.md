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
  - Provider definitions with token references (resolved from file paths or environment variables)
  - Organization list for auto-discovery of repositories
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
Contains provider and top-level configuration:
```yaml
gpg_key_path: '.secure_files/autoupdate.asc'
exclude_forks: true

providers:
  - type: 'github'
    token: '.secure_files/github_access_token.key'
    organizations:
      - 'rios0rios0'
```

#### .github/workflows/autoupdate.yaml
GitHub Actions workflow that:
- Runs daily at 09:00 UTC / 6:00 AM BRT (`cron: '0 9 * * *'`)
- Can be manually triggered via workflow_dispatch
- Downloads latest autoupdate binary using the official install script
- Writes secrets to `.secure_files/` (GPG key and GitHub token as files)
- Validates the GPG key before running
- Runs `./autoupdate run` (reads credentials from file paths in `.autoupdate.yaml`)
- Cleans up `.secure_files/` on completion (always, even on failure)

### Workflow Secrets Required
The GitHub Actions workflow expects these repository secrets:
- `PERSONAL_ACCESS_TOKEN` - GitHub token with repo access (written to `.secure_files/github_access_token.key`)
- `GPG_PRIVATE_KEY` - Armored GPG private key for commit signing (written to `.secure_files/autoupdate.asc`)

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
- **Missing secrets**: Workflow will fail if `PERSONAL_ACCESS_TOKEN` or `GPG_PRIVATE_KEY` secrets are not configured

### Making Changes
- **ALWAYS** validate configuration changes with `yamllint .autoupdate.yaml`
- **ALWAYS** test autoupdate can parse the config with `./autoupdate run --dry-run`
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
