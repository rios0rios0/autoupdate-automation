<h1 align="center">Autoupdate Automation</h1>
<p align="center">
    <a href="https://github.com/rios0rios0/autoupdate-automation/releases/latest">
        <img src="https://img.shields.io/github/release/rios0rios0/autoupdate-automation.svg?style=for-the-badge&logo=github" alt="Latest Release"/></a>
    <a href="https://github.com/rios0rios0/autoupdate-automation/blob/main/LICENSE">
        <img src="https://img.shields.io/github/license/rios0rios0/autoupdate-automation.svg?style=for-the-badge&logo=github" alt="License"/></a>
</p>

GitHub Actions automation workflow that runs [AutoUpdate](https://github.com/rios0rios0/autoupdate) on a schedule to automatically discover and upgrade dependencies across repositories.

Take a look at the [Autoupdate repository](https://github.com/rios0rios0/autoupdate) for more information about how the tool works.

## Configuration

The daily workflow runs one job per owner. Each owner needs its own fine-grained PAT, because a
GitHub fine-grained token is bound to a single resource owner and cannot span several:

| Owner         | Secret                     |
|---------------|----------------------------|
| `rios0rios0`  | `PERSONAL_ACCESS_TOKEN`    |
| `medhub-life` | `MEDHUB_ACCESS_TOKEN` |
| `prefy`       | `PREFY_ACCESS_TOKEN`       |

Every token's lifetime must be **366 days or less** — the organizations reject longer-lived
fine-grained tokens with a `403`. Commit signing additionally needs the `GPG_PRIVATE_KEY` secret
and the `GIT_USER_NAME`, `GIT_USER_EMAIL` and `GIT_USER_SIGNINGKEY` variables.

To cover another owner, add an entry to `strategy.matrix.owner` in
[`.github/workflows/autoupdate.yaml`](.github/workflows/autoupdate.yaml) and create its secret.

## Contributing

Contributions are welcome. See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## License

See [LICENSE](LICENSE) for details.
