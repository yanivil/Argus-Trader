# Contributing to Argus-Trader

The day-to-day working agreement. The standard behind it, with the reasoning and
the provenance of each rule, is [`docs/git-hardening-guidelines.md`](docs/git-hardening-guidelines.md);
this file is the practical how-to.

> **This repository is public.** Argus-Trader trades real capital through keyed,
> paid services. Nothing that identifies an account, a position, or a credential
> belongs in a commit — including in a branch that is later deleted, because git
> history is not a delete-capable store.

---

## 1. The flow — every change, no exceptions

1. **Branch from `main`:** `<type>/<short-topic>` — `feat/tier0-regime-engine`,
   `fix/zigzag-atr-window`, `docs/phase2-design`, `ci/pin-actions`.
2. **One topic per branch.** A PR that does two things gets reviewed as well as
   the reviewer's patience for the second one.
3. **Open a PR** and fill in all four sections of the template: Context / Why,
   Summary of Changes, Test Steps, Doc Impact. CI fails the PR if a heading is
   missing.
4. **PR title is a Conventional Commit** — `type(scope): summary`. This
   repository squash-merges, so the title becomes the commit message on `main`.
   CI checks it.
5. **CI green before merge.** Until the branch ruleset is configured the merge
   button stays clickable on a red PR — treat red as blocking anyway.
6. **Squash-merge**, then delete the branch.

## 2. Rules that matter

- **Never push directly to `main`.** The *Main guard* workflow opens an incident
  issue for any push to `main` that did not arrive through a merged PR, and for
  any force-push. Treat that issue as a P1: **revert first, discuss after.**
- **Never force-push `main`.** A bad merge is undone with a revert PR, never by
  rewriting history.
- **`CHANGELOG.md` gets an entry in the same PR**, under `[Unreleased]`, in
  Keep a Changelog format. CI enforces this. If an entry genuinely does not
  apply, apply the `skip-changelog` label — the label is visible on the PR, so
  the exemption is a recorded decision rather than a silent one.
- **Tests ship with the code.** Any PR that adds or changes behaviour includes
  its tests. "N/A" is for docs and config only.
- **Bug fixes are red → green.** The first commit on the branch is a failing test
  that reproduces the bug. The PR records the root cause, why the existing tests
  missed it, and the class-level prevention added — not just a test for the one
  case.
- **Doc comments on public surface.** Every public function, class, and module
  gets a docstring: purpose, parameters, returns, raises. Reviewers push back on
  undocumented public surface.
- **Comments explain the constraint, not the mechanics.** Every threshold and
  tuning constant carries a one-line reason for its value — the CI regression
  gate (design §10) makes those values load-bearing, and a number nobody can
  justify is a number nobody can safely change.
- **Reviews are real.** Approving means you read it and would maintain it.

## 3. Secrets and data

- Real values live in GitHub Actions secrets or the local environment, never in
  the repository. Required variables are documented in `.env.example`.
- `.gitignore` excludes credentials **by pattern** (`*.key`, `*secrets*`,
  `*credentials*`, `.env*`) rather than by filename, so a file nobody
  anticipated is still caught by shape.
- If a credential is ever pushed, **rotate it** — removing the commit does not
  make the key safe.
- Report a vulnerability privately through the Security tab, never as a public
  issue. See [`SECURITY.md`](SECURITY.md).

## 4. Dependencies

- Generate `requirements.txt` with `pip-compile --generate-hashes` and install
  with `pip install --require-hashes`. A version pin trusts the package index to
  serve the same artifact tomorrow; a hash pin does not.
- Pin every GitHub Action to a full 40-character commit SHA with the version in
  a trailing comment. Verify the SHA against upstream rather than copying it:

  ```bash
  git ls-remote --tags https://github.com/actions/checkout | grep 'refs/tags/v7.0.1'
  ```

- Dependabot opens the PRs that move those pins forward, weekly.

## 5. CI at a glance

| Job | Runs on | What it does | Blocking |
|---|---|---|---|
| `pr-quality` | every PR | PR title is a Conventional Commit; body has the four required sections | yes |
| `changelog` | every PR | `CHANGELOG.md` changed in this PR, unless labelled `skip-changelog` | yes |
| `tests` | every PR | Lint and tests once `src/` and `tests/` exist; reports honestly until then | yes |
| `audit` | every PR | `pip-audit` against the pinned set once `requirements.txt` exists | advisory |
| `Main guard` | every push to `main` | Opens an incident issue for a direct push or a force-push | n/a |

Two more workflows are due when implementation code lands, per the design:
`backtest_regression_gate.yml` (design §10 — blocks a PR that degrades take-profit
rate, expectancy, profit factor, or drawdown against the committed baseline) and
`daily_batch.yml` (design §11). The daily batch **must** commit as
`github-actions[bot]`; that is the actor the Main guard exempts, and a batch job
committing under a human identity would raise a false incident every morning.

## 6. Release and rollback

No release process exists yet — Phase 1 is design only, and the repository has no
tags. When implementation lands, the flow follows the standard in
[`docs/git-hardening-guidelines.md`](docs/git-hardening-guidelines.md) §8: a
manual `workflow_dispatch` semver bump that tags `vX.Y.Z` and publishes release
notes generated from merged PRs.

Roll back a bad change with the **Revert** button on the merged PR, which opens a
revert PR to review and merge. Never force-push `main` to undo anything.
