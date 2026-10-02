# Contributing to SelfHostHub

Thanks for your interest in improving SelfHostHub! This guide covers how to get set up,
the workflow for changes, and the conventions we follow.

By contributing, you agree that your contributions are licensed under the project's
[GPL-3.0](LICENSE) license.

## Getting set up

The project is in the design phase. Once code lands, you'll need:

- **Go** (latest stable) for `agent/`, the service that runs inside WSL
- **Rust + Node** for `app/`, the Tauri Windows app
- **Windows 10 22H2 or 11 with WSL2** for end-to-end testing. The agent alone runs on any Linux machine with Docker.

## Before you start

- **Search existing issues** before filing a new one, and comment on an issue before starting
  significant work so we can avoid duplicate effort.
- For anything beyond a small fix, open an issue to discuss the approach first.
- Found a security vulnerability? **Do not open a public issue** — follow [SECURITY.md](SECURITY.md).

## Workflow

**All changes go through a branch and a pull request — no direct pushes to `main`.**

1. Fork the repo (external contributors) or create a branch (maintainers).
2. Branch naming: `feat/<slug>`, `fix/<slug>`, `chore/<slug>`, `docs/<slug>`.
3. Make your change with tests.
4. Run the checks below locally.
5. Open a PR with a clear description of **what** changed and **why**. Link the issue it
   closes (`Closes #123`).
6. Keep PRs focused — one logical change per PR is much easier to review.

### Checks to run before opening a PR

```bash
cd agent && go vet ./... && go test ./...     # agent
cd app && npm run lint && npm test            # Windows app UI
cd app/src-tauri && cargo clippy && cargo test
```

### Keeping the docs current

Docs live in [`docs/`](docs/). A `feat:` PR must add or update a page explaining what the
feature does, who it's for and how to use it, in the same PR. Write for non-technical users first.

### Releasing

Releases are manual — merging PRs never releases. Run **Actions → Release → Run workflow** to
open or update the `chore(main): release x.y.z` PR, merge it when ready, then run the workflow
again to tag `vX.Y.Z` and create the GitHub release with a signed source tarball.

### Commit messages

Use [Conventional Commits](https://www.conventionalcommits.org/): `type(scope): subject`
(e.g. `fix(agent): retry DuckDNS update on timeout`). Keep the subject imperative and under
~50 characters; add a body only when the *why* isn't obvious.

## Adding an app to the catalog

Each app is one YAML manifest in `catalog/`. A new app must:

- run in **one container** (or explain why not in the PR),
- work on the home network **without internet** once installed,
- pin an exact image version (never `latest`),
- pass the CI install + health check.

## Code style

- **Plain English in every user-facing string.** No jargon ("container", "WSL", "port") in the
  dashboard. Write "Replace YNAB", not "Install actual-server".
- **Every error tells the user what to do next**, with a button where possible.
- **Never delete user data without explicit confirmation**, and default to keeping it.
- Go: `gofmt`, small packages, errors wrapped with context. Rust: `clippy` clean.
- Match the style of the surrounding code.

## Reporting bugs & requesting features

Open a GitHub issue with:
- What you expected vs. what happened
- Steps to reproduce (for bugs)
- Version / commit and environment details where relevant

## Code of Conduct

Participation is governed by our [Code of Conduct](CODE_OF_CONDUCT.md). Please read it.
