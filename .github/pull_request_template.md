<!-- ===========================================================================
     PR title must follow Conventional Commits:  type(scope): summary
       feat(tier1): ATR-adaptive ZigZag extrema
       fix(governance): the arbitration tie-break was reading the bear score twice
       docs(design): restore the §6 position-sizing formulas

     This repository squash-merges, so the title above becomes the commit
     message on main. Write it for `git log`, not for the reviewer.

     The four headings below are checked by CI (.github/workflows/ci.yml).
     Do not rename or remove them. The standard is docs/git-hardening-guidelines.md.
=========================================================================== -->

## Context / Why

<!-- The trigger and the reason. A reader a year from now must understand this
     PR without the conversation that produced it. Link the design section
     (§N) or the issue it comes from. -->

## Summary of Changes

<!-- What actually changed, per file or per area. Enough that a reviewer knows
     where to look before opening the diff. -->

## Test Steps

<!-- What you ran, and what a reviewer should run to confirm it.
     For a docs-only change, say so and give the render or link check you did.
     For a behaviour change, name the tests that cover it. -->

## Doc Impact

<!-- Which docs moved in this same commit, or an explicit statement that none
     needed to. "None — internal refactor, no documented behaviour changed" is
     a valid answer; silence is not. -->

---

## Checklist

- [ ] One topic. This PR does one thing.
- [ ] `CHANGELOG.md` has an entry under `[Unreleased]` (Keep a Changelog format).
- [ ] New or changed behaviour ships with its tests in this PR — or N/A for docs and config.
- [ ] Public functions and classes have doc comments (purpose, params, returns, raises).
- [ ] Comments explain constraints and non-obvious reasoning, not the obvious.
- [ ] No secrets, credentials, or personal data in code, config, fixtures, or logs. **This repository is public.**
- [ ] Any new GitHub Action is pinned to a full commit SHA with the version in a trailing comment.
- [ ] Any changed threshold or tuning constant carries a one-line reason for its value.

## Bug fix only — escape analysis

<!-- Delete this section if the PR is not a bug fix. -->

- **Root cause:**
- **Why existing tests missed it** (missing level / wrong fixture / untested edge / environment gap / wrong assumption):
- **Class-level prevention added** (a test family or property, not just the single case):
- [ ] The first commit on the branch is the failing repro (red → green).
