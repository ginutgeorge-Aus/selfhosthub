<!-- Keep PRs small and single-purpose. Branch naming: feat/<slug>, fix/<slug>, chore/<slug>. -->

## What & why

<!-- One or two lines: what this changes and the reason. Link issues: "Closes #123, closes #124" (repeat the keyword per issue). -->

## Changes

-

## Checklist

- [ ] Tests pass and lint is clean for every touched part (`agent/`: `go vet` + `go test`; `app/`: lint + tests + `cargo clippy`)
- [ ] PR title follows Conventional Commits (`feat:`, `fix:`, …). It becomes the CHANGELOG entry.
- [ ] User-facing text is plain English (no "container", "WSL", "port") and every error says what to do next
- [ ] No user data is deleted without confirmation
- [ ] **New catalog app?** single container, works offline, pinned image version, CI install + health check passes
- [ ] **Windows-side change?** tested on a real Windows 10 22H2 or 11 laptop (GitHub runners can't run WSL2)
- [ ] **Feature?** docs page added or updated in `docs/` (what, who, how)
- [ ] No secrets, tokens or real DuckDNS/deSEC credentials in the diff
- [ ] **Bugfix?** a failing-first regression test ships in this PR (guardrail ladder L3)
- [ ] **Same class of bug twice?** promoted to a guardrail (lint rule or CI gate, L4–5), not just a note

## Notes for reviewer

<!-- Anything to focus on, known gaps, or follow-ups filed as issues. -->
