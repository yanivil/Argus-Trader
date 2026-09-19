# Git Hardening Guidelines

The repository standard for `yanivil/Argus-Trader`, distilled from the controls already in
force across the other seven repositories in this account.

Nothing here is aspirational or imported from an external framework. Every rule below is
written from a control that exists in at least one of those repositories today, and each rule
names where it came from so it can be checked against the source.

---

## 0. Evidence base

Read on 2026-09-19, at the head of each default branch:

| Repository | Visibility | What it contributed |
|---|---|---|
| `SignalSync` | public | The workflow-hardening standard: SHA-pinned actions, least-privilege `permissions`, per-job timeouts, concurrency groups, hash-checked dependency installs |
| `TASE-125` | public | The widest secret-ignore set, and a `SECURITY.md` that states a zero-credential architecture and an explicit in/out-of-scope boundary |
| `travel-planner` | public | Agent-facing working agreements in `CLAUDE.md`, decision and regression logs, the public-repo personal-data rule, matrix CI |
| `capital-intelligence-network` | private | `pip-audit` as a separate non-gating CI job; the no-direct-push rule written for agents in `.agents/AGENTS.md` |
| `family-finance-organizer` | private | `.gitignore` as a data-classification tool: local financial data and databases named and excluded with a stated reason |
| `referent-ai` | private | The governance layer: machine-checked PR quality, the `main` tripwire, the manual semver release flow, and the written working agreement |
| `STRIKE` | private | Conventional Commits held consistently across a long docs-and-code history |
| `Argus-Trader` | public | The PR body shape already used on PRs #1–#4 |

Two facts about this estate shaped the rules:

- **Not one repository uses a client-side hook.** There is no `.husky/`, no `lefthook.yml`, no
  `.pre-commit-config.yaml`, no `commitlint.config.js`, and no `CODEOWNERS` anywhere. Enforcement
  is entirely server-side, in GitHub Actions. These guidelines keep that choice: a control a
  contributor can skip with `--no-verify` is not a control.
- **Coverage is uneven.** `SignalSync` is the only repository where every workflow is pinned,
  scoped and timed out; `referent-ai` is the only one that machine-checks PR quality and watches
  `main`. The strongest control found anywhere is the one written down here.

---

## 1. Branch and merge discipline

**1.1 `main` is never written to directly.** Every change — code, docs, configuration, workflow —
lands on a branch and merges through a pull request.
*Stated in three repositories independently: `referent-ai/CONTRIBUTING.md` ("Never push directly
to `main`"), `travel-planner/CLAUDE.md` ("Never commit directly to `main`"), and
`capital-intelligence-network/.agents/AGENTS.md` ("NEVER push changes directly to the `main`
branch").*

**1.2 Branch names are `<type>/<short-topic>`.** The type is the same vocabulary as the commit
type: `feat/`, `fix/`, `docs/`, `chore/`, `ci/`, `test/`, `refactor/`, `perf/`.
*From `referent-ai/CONTRIBUTING.md` and `travel-planner/CLAUDE.md`; visible in the merge history
of `SignalSync` (`feat/double-bottom`, `feat/site-open-trades`) and `family-finance-organizer`
(`fix/fibi-header-spelling`, `feat/card-fixed-tab`).*

**1.3 One topic per pull request.**
*From `travel-planner/CLAUDE.md`: "Keep PRs reviewable: one topic per PR."*

**1.4 `main` history is append-only. Never force-push it.** A bad merge is undone by a revert
pull request, or by redeploying the previous tag — never by rewriting history.
*From `referent-ai/CONTRIBUTING.md`: "Never force-push `main` to 'undo' anything."*

**1.5 Merge strategy is fixed per repository and written down.** Two strategies are in use in
this estate and both are defensible; what is not defensible is drifting between them, because the
merge strategy decides what `git log main` is worth reading. `referent-ai` and
`capital-intelligence-network` squash, so `main` carries one Conventional Commit per PR.
`SignalSync`, `travel-planner` and `family-finance-organizer` keep merge commits, so `main`
carries the branch's individual commits under a `Merge pull request #N` cap.

**For Argus-Trader: squash.** The first four PRs each landed several commits of one editorial
revision, and squashing keeps one reviewed change as one entry on `main`. The PR title therefore
becomes the commit message on `main` and is held to §2.1 by CI.

---

## 2. Commit and pull request conventions

**2.1 Conventional Commits, `type(scope): summary`.** Allowed types: `feat`, `fix`, `docs`,
`test`, `refactor`, `perf`, `build`, `ci`, `chore`, `style`, `revert`. With squash merging the PR
title is the commit message, so the title is what CI checks.
*From `referent-ai/.github/workflows/ci.yml`, which enforces exactly this regex, and from the
commit history of `STRIKE`, `capital-intelligence-network` and `family-finance-organizer`.*

**2.2 Every pull request body carries four sections**, and CI fails the PR if any heading is
missing:

| Heading | What belongs in it |
|---|---|
| `## Context / Why` | The trigger and the reason. A reader must not need the chat that produced the PR. |
| `## Summary of Changes` | What changed, per file or per area. |
| `## Test Steps` | What was run, and what a reviewer should run to confirm it. |
| `## Doc Impact` | Which docs moved in the same commit, or an explicit statement that none needed to. |

*The headings are the ones already used on `Argus-Trader` PRs #1–#4. The mechanism — a CI job
that greps the PR body for required headings and fails with an `::error::` annotation — is taken
from `referent-ai/.github/workflows/ci.yml`.*

**2.3 Bug-fix pull requests carry an escape analysis.** Root cause, why the existing tests missed
it, and the class-level prevention added — not just a test for the single case. The first commit
on the branch is the failing repro.
*From `travel-planner/CLAUDE.md` and its PR template; `travel-planner` logs each escape in
`REGRESSIONS.md` as `R-NNN`.*

**2.4 Automated commits use the bot identity.** Any workflow that commits uses
`github-actions[bot]` with `41898282+github-actions[bot]@users.noreply.github.com`. This is not
cosmetic: the `main` tripwire in §3.2 exempts that actor, so a batch job that commits under a
human identity raises a false incident.
*From `SignalSync`'s `daily-scan.yml` and `sync-wiki.yml`.*

---

## 3. Guarding `main`

**3.1 Branch protection is the primary control where the plan allows it.** `Argus-Trader` is
public, so on GitHub Free the ruleset is available and must be configured — see §8. This is the
one place `Argus-Trader` can go further than `referent-ai`, which is private and was therefore
built around the absence of branch protection.

**3.2 A tripwire backs it up.** A workflow on `push: [main]` opens an incident issue for any push
whose head commit is not associated with a merged pull request, and for any force-push. It is not
redundant with branch protection: a repository admin can bypass a ruleset, and the tripwire is
what makes that bypass visible afterwards.
*Ported from `referent-ai/.github/workflows/main-guard.yml`, which exists precisely because
branch protection is unavailable on private repositories on the Free plan. The Argus port uses
the `gh` CLI instead of `actions/github-script`, so the tripwire depends on no third-party
action at all.*

**3.3 An incident issue is a P1: revert first, discuss after.**
*From `referent-ai/CONTRIBUTING.md`.*

**3.4 Red CI means do not merge, whether or not the button is clickable.**
*From `referent-ai/CONTRIBUTING.md`: "On the Free plan checks are advisory (the merge button
stays clickable) — treat them as required anyway."*

---

## 4. Workflow hardening

Every rule in this section is taken from `SignalSync`, the only repository in the estate where
all seven workflows satisfy all of them.

**4.1 Declare `permissions` at the top of every workflow, starting from `contents: read`.**
Elevate only the specific scope a job needs, and leave a comment saying why.
*`SignalSync/daily-scan.yml`: `contents: write  # needed to commit output/ back to the repo`.
`travel-planner/ci.yml` makes the same point the other way: `# least privilege: no job needs to
write to the repo (D-024)`.*

**4.2 Pin third-party actions to a full 40-character commit SHA, with the version in a trailing
comment.** A tag is a movable pointer; a compromised upstream can repoint `v4` without any change
landing here.

```yaml
- uses: actions/checkout@3d3c42e5aac5ba805825da76410c181273ba90b1 # v7.0.1
```

*Every `uses:` in `SignalSync` is pinned this way. `TASE-125`, `travel-planner`,
`capital-intelligence-network` and `referent-ai` all still use floating tags — that is the single
largest gap in the estate, and the reason this rule is stated before the others.*

**When adding or bumping a pin, verify the SHA against upstream rather than copying it:**

```bash
git ls-remote --tags https://github.com/actions/checkout | grep 'refs/tags/v7.0.1'
```

**4.3 Every job sets `timeout-minutes`.** An unbounded job is a runner held until GitHub's
six-hour ceiling.
*Present on every job in `SignalSync` and `capital-intelligence-network`; absent from
`TASE-125`, `travel-planner` and `referent-ai`.*

**4.4 Never interpolate untrusted input into a `run:` script.** A pull request title, body, or
branch name is attacker-controlled on a public repository. Pass it through `env:` and read it as a
shell variable, so its content can never be parsed as script.

```yaml
- env:
    TITLE: ${{ github.event.pull_request.title }}
  run: |
    [[ "$TITLE" =~ $regex ]] || exit 1      # safe: $TITLE is data, never code
```

*From `referent-ai/.github/workflows/ci.yml` and `SignalSync/sync-wiki.yml`, which passes
`secrets.GITHUB_TOKEN` through `env: TOKEN` rather than inlining it.*

**4.5 Use a `concurrency` group on anything stateful or expensive.** `cancel-in-progress: true`
where a newer run supersedes an older one; `false` where runs must not overlap because they write
shared state.
*From `SignalSync`: `pages.yml` cancels, `daily-scan.yml` does not.*

**4.6 Gate deployment and publication steps on the branch.**
*From `SignalSync/pages.yml`: `if: github.ref == 'refs/heads/main'` — a branch push builds the
site without deploying it.*

**4.7 Upload diagnostics on failure, with a retention limit.**
*From `travel-planner/ci.yml`, which uploads the Playwright report `if: failure()` with
`retention-days: 7`.*

---

## 5. Supply chain

**5.1 Dependabot watches both `github-actions` and the language ecosystem, weekly.** Watching
`github-actions` is what keeps §4.2's SHA pins from going stale into unpatched versions — pinning
without Dependabot trades a supply-chain risk for an unpatched-dependency risk.
*From `SignalSync/.github/dependabot.yml`, whose comment states this directly; also in
`TASE-125`, `travel-planner` and `referent-ai`.*

**5.2 Pin dependencies by hash, and install with the hash check enforced.** Generate with
`pip-compile --generate-hashes` and install with `pip install --require-hashes`. A version pin
trusts the index to serve the same artifact tomorrow; a hash pin does not.
*From `SignalSync`, the only repository doing this: `requirements.txt` is a `pip-compile
--generate-hashes` output and every install step passes `--require-hashes`.*

**5.3 Audit dependencies for known vulnerabilities in CI.** Run it as a separate job so a new CVE
in a transitive dependency reports without blocking an unrelated merge.
*From `capital-intelligence-network/.github/workflows/tests.yml`, which runs `pip-audit -r
requirements.txt --desc on` in its own `audit` job; `travel-planner` runs `npm audit
--audit-level=high` inline.*

**5.4 Commit the lockfile.**
*From `travel-planner/CLAUDE.md`.*

---

## 6. Secret and data hygiene

**6.1 Secrets never enter the repository — not once, not in a branch, not in a reverted commit.**
Git history is not a delete-capable store: a key that was pushed must be rotated, not removed.
Required variables are documented in `.env.example`; real values live only in GitHub Actions
secrets or the host vault.
*From `travel-planner/CLAUDE.md`, whose `.gitignore` encodes the exception `!.env.example`.*

**6.2 `.gitignore` excludes secrets by pattern, not by filename.** The widest set in the estate,
worth copying verbatim:

```gitignore
.env
.env.*
*.env
*.pem
*.key
*.cert
*.crt
*.pfx
*.p12
*.token
*secrets*
*credentials*
credentials.json
client_secret*.json
```

*From `TASE-125/.gitignore`, under the heading "Sensitive & Secret Files (Zero-Secret
Architecture)".*

**6.3 `.gitignore` also carries the data classification, with the reason in a comment.** Naming
the class of data and why it is excluded turns the file into the record of a decision, which
survives the person who made it.
*From `family-finance-organizer/.gitignore`: "local financial data — NEVER commit (DB lives in
~/.family-finance by default, but guard against any copy landing inside the repo)". From
`travel-planner/.gitignore`: "local personal fixtures — real trip data never enters this public
repo".*

**6.4 A public repository carries an explicit no-personal-data rule.**
*From `travel-planner/CLAUDE.md`: "This repo is PUBLIC. Never commit personal data: real family
names, phone numbers, ticket PDFs, or the user's personal trip files."*

**6.5 `SECURITY.md` states the credential posture, the reporting channel, and the boundary.**
Private vulnerability reporting through GitHub's Security tab, never a public issue; an explicit
in-scope and out-of-scope list so a report about strategy profitability is not treated as a
vulnerability.
*From `TASE-125/SECURITY.md` and `SignalSync/SECURITY.md`, whose scope sections both exclude
trading outcomes and third-party data providers.*

**6.6 Secret scanning and push protection are enabled server-side.** Push protection is the only
control here that stops a leak before it reaches the remote; the rest only limit the damage.
*Stated as enabled in `TASE-125/SECURITY.md`. See §8 — it is a repository setting, not a file.*

---

## 7. Documentation moves in the same commit

**7.1 `CHANGELOG.md` gets an entry in the same pull request**, under `[Unreleased]`, in
Keep a Changelog format.
*In `SignalSync`'s PR checklist and `travel-planner/CLAUDE.md`; `Argus-Trader` already keeps this
format.*

**7.2 Public functions and classes carry doc comments.** Reviewers push back on undocumented
public surface.
*From `referent-ai/CONTRIBUTING.md` and `travel-planner/CLAUDE.md`.*

**7.3 Comments explain the constraint, not the mechanics.**
*From `travel-planner/CLAUDE.md`: comments "explain constraints and non-obvious whys ... never
narrate the obvious." `SignalSync` applies it to tuning constants — every threshold carries a
one-line reason, and its PR checklist enforces that for changed thresholds.*

**7.4 Significant decisions are logged with a stable identifier, in the PR that makes them.**
Context, decision, why, and when to revisit. A PR that changes an earlier decision edits that
entry rather than silently diverging.
*From `travel-planner`, which keeps `DECISIONS.md` (`D-xxx`) and `REGRESSIONS.md` (`R-NNN`) and
cites the IDs in commit subjects: "fix: serialize store mutations — the coalescing chain must
never race (D-031)".*

---

## 8. Server-side settings

These are repository settings, not files. They cannot be committed and must be set in the GitHub
UI by the repository owner. They are listed here because §1.1, §3.1 and §6.6 are only partly
enforced without them.

**Settings → Branches → Add branch ruleset**, targeting `main`:

- Require a pull request before merging
- Require status checks to pass: `pr-quality`, `changelog`, `tests`
- Require branches to be up to date before merging
- Block force pushes
- Restrict deletions

**Settings → General → Pull Requests:** allow squash merging only, set the squash commit message
to "pull request title", and enable automatic branch deletion after merge.
*The delete-on-merge behaviour is documented in `referent-ai/CONTRIBUTING.md`.*

**Settings → Code security:** enable secret scanning, push protection, and Dependabot security
updates. *`SignalSync`'s `dependabot.yml` notes that security updates are enabled separately in
repository settings and that the file adds version-update PRs on top.*

**Settings → Code security → Private vulnerability reporting:** enable, so `SECURITY.md` §6.5
points at a channel that exists.

---

## 9. Conformance across the estate

Read at each repository's default branch head on 2026-09-19. `—` means the repository has no
`.github/` directory at all.

| Control | SignalSync | TASE-125 | travel-planner | capital-intel | family-finance | referent-ai | STRIKE | Argus (before) |
|---|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|
| CI runs on every PR | ✅ | ✅ | ✅ | ✅ | — | ✅ | — | ❌ |
| `permissions:` block declared | ✅ | ❌ | ✅ | ❌ | — | ✅ | — | ❌ |
| Actions pinned to SHA | ✅ | ❌ | ❌ | ❌ | — | ❌ | — | ❌ |
| `timeout-minutes` on every job | ✅ | ❌ | ❌ | ✅ | — | ❌ | — | ❌ |
| Dependabot configured | ✅ | ✅ | ✅ | ❌ | ❌ | ✅ | ❌ | ❌ |
| Dependency CVE audit in CI | ❌ | ❌ | ✅ | ✅ | — | ❌ | — | ❌ |
| Hash-pinned dependencies | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| PR template | ✅ | ❌ | ✅ | ❌ | ❌ | ✅ | ❌ | ❌ |
| PR quality machine-checked | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ | ❌ |
| `main` push tripwire | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ | ❌ |
| Written no-direct-push rule | ✅ | ❌ | ✅ | ✅ | ❌ | ✅ | ❌ | ❌ |
| `SECURITY.md` | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |
| `.env` ignored | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ |

No single repository satisfies every row. This document is the union of the strongest column in
each row, and §10 is that union applied to `Argus-Trader`.

---

## 10. Adoption checklist

Committed in this repository:

- [x] `.gitignore` — §6.2, §6.3
- [x] `.github/pull_request_template.md` — §2.2
- [x] `.github/workflows/ci.yml` — §2.1, §2.2, §4.1–§4.4, §5.3, §7.1
- [x] `.github/workflows/main-guard.yml` — §3.2
- [x] `.github/dependabot.yml` — §5.1
- [x] `CONTRIBUTING.md` — §1, §2
- [x] `SECURITY.md` — §6.5
- [x] `docs/git-hardening-guidelines.md` — this document

Requires the repository owner, in the GitHub UI — §8:

- [ ] Branch ruleset on `main`: require PR, require `pr-quality` / `changelog` / `tests`, block force pushes, restrict deletions
- [ ] Squash-merge only; squash message = PR title; auto-delete merged branches
- [ ] Secret scanning, push protection, Dependabot security updates
- [ ] Private vulnerability reporting

Due when the first implementation code lands:

- [ ] `requirements.txt` generated by `pip-compile --generate-hashes`, installed with `--require-hashes` — §5.2
- [ ] `pip-audit` job activates automatically once `requirements.txt` exists — already wired in `ci.yml`
- [ ] `.github/workflows/backtest_regression_gate.yml`, per design §10, blocking on TP-rate, expectancy, profit-factor and drawdown regression against `test_results/benchmark_latest.json`
- [ ] `.github/workflows/daily_batch.yml`, per design §11, committing as `github-actions[bot]` so the §3.2 tripwire stays quiet
