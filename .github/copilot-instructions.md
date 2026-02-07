# Autoupdate Automation Repository

This repository provides automated dependency and version management across multiple projects using the [autoupdate tool](https://github.com/rios0rios0/autoupdate). It is a configuration and automation repository containing a GitHub Actions workflow that runs the autoupdate tool daily to detect outdated dependencies and create Pull Requests to upgrade them.

**ALWAYS** reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

### Bootstrap and Test the Repository
- Install required system dependencies:
  - `sudo apt-get update && sudo apt-get install -y unzip curl jq` -- takes 30-60 seconds
- Download and test autoupdate binary:
  - `export AUTOUPDATE_VERSION=$(curl -sSLH 'Accept: application/vnd.github.v3+json' 'https://api.github.com/repos/rios0rios0/autoupdate/releases/latest' | jq -r '.tag_name')`
  - `curl -LO "https://github.com/rios0rios0/autoupdate/releases/download/$AUTOUPDATE_VERSION/autoupdate-$AUTOUPDATE_VERSION.zip"` -- downloads the binary
  - `unzip "autoupdate-$AUTOUPDATE_VERSION.zip"` -- extracts in <1 second
  - `./autoupdate --help` -- verify binary works

### Configuration Management
- The main configuration file is `.autoupdate.yaml` containing:
  - Provider definitions with token references (resolved from environment variables or file paths)
  - Organization list for auto-discovery of repositories
  - Updater settings (Terraform and Go dependency updaters)
- Token format supports inline values, `${ENV_VAR}` references, or file paths
- Validate configuration syntax: `yamllint .autoupdate.yaml` -- takes <1 second
  - WARNING: yamllint will report missing document start "---" which is acceptable

### Testing Workflow Components
- Test autoupdate processing:
  - `./autoupdate run --dry-run` -- preview changes without creating PRs
  - With valid `GITHUB_TOKEN` env var: discovers repos and scans dependencies
  - Without credentials: fails with authentication errors (<1 second), which is expected

## Validation

### Manual Workflow Testing
- **ALWAYS** test workflow components after making configuration changes
- Run the complete workflow simulation:
  1. Download dependencies (30-60 seconds)
  2. Download autoupdate binary (<1 second)
  3. Test binary execution (<1 second)
  4. Validate configuration syntax (<1 second)
- **NEVER CANCEL** workflow operations - all steps complete in under 2 minutes
- The actual GitHub Actions workflow runs daily at 12:00 UTC and can be manually triggered

### Configuration Validation Steps
- Validate YAML syntax: `yamllint .autoupdate.yaml`
- Test autoupdate config parsing: `./autoupdate run --dry-run` (expect authentication errors without credentials)
- Verify provider organizations are accessible
- Check that token references (`${GITHUB_TOKEN}`) correspond to configured repository secrets

## Common Tasks

### Repository Structure
```
.
├── .autoupdate.yaml         # Main autoupdate configuration (providers + updaters)
├── .editorconfig            # Editor configuration
├── .github/
│   ├── workflows/
│   │   └── autoupdate.yaml  # Daily automation workflow
│   ├── copilot-instructions.md
│   └── pull_request_template.md
└── README.md                # Basic project description
```

### Key Configuration Files

#### .autoupdate.yaml
Contains provider and updater configuration:
```yaml
providers:
  - type: github
    token: "${GITHUB_TOKEN}"
    organizations:
      - "rios0rios0"

updaters:
  terraform:
    enabled: true
    auto_complete: false
    target_branch: "main"
  golang:
    enabled: true
    target_branch: "main"
```

#### .github/workflows/autoupdate.yaml
GitHub Actions workflow that:
- Runs daily at 12:00 UTC (`cron: '0 12 * * *'`)
- Can be manually triggered via workflow_dispatch
- Downloads latest autoupdate binary from GitHub releases
- Runs `./autoupdate run` with `GITHUB_TOKEN` from secrets

### Workflow Secrets Required
The GitHub Actions workflow expects these repository secrets:
- `PERSONAL_ACCESS_TOKEN` - GitHub token with repo access (mapped to `GITHUB_TOKEN` env var)

### Expected Timing
- **apt-get update && install**: 30-60 seconds
- **autoupdate download**: <1 second
- **autoupdate extraction**: <1 second
- **configuration validation**: <1 second
- **Complete workflow test**: <2 minutes total

### Common Failure Scenarios
- **"at least one provider must be configured"**: Check `.autoupdate.yaml` has valid provider entries
- **"providers[0].token is required"**: Ensure `GITHUB_TOKEN` env var is set or token is configured
- **"providers[0].organizations must have at least one entry"**: Add at least one organization
- **Missing secrets**: Workflow will fail if `PERSONAL_ACCESS_TOKEN` secret is not configured

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
