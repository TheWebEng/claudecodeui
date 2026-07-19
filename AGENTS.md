# CloudCLI Web.Eng Contribution Plan

This repository is the Web.Eng organization fork of
[`siteboon/claudecodeui`](https://github.com/siteboon/claudecodeui).

It has two jobs:

1. Let Web.Eng use valuable improvements before upstream has reviewed and released them.
2. Send those improvements upstream in focused, maintainable pull requests.

The fork is not intended to become a separate product. Our default decision is to build only changes that solve a general CloudCLI problem and would benefit upstream users. Keeping the fork close to upstream is more valuable than accumulating organization-specific behavior.

## Instruction priority

- Follow the repository's `CONTRIBUTING.md` for all work, especially outbound pull requests.
- Follow the Web.Eng workspace instructions in `webeng-team-core/AGENTS.md`.
- This file adds the branch, development, integration, and release rules for this fork.
- If upstream changes its contribution requirements, update this file rather than working around them.

## Repository ownership and remotes

- Organization repository: `https://github.com/TheWebEng/claudecodeui`
- Official upstream: `https://github.com/siteboon/claudecodeui`
- `origin` must point to `TheWebEng/claudecodeui`.
- `upstream` must point to `siteboon/claudecodeui`.

Verify before branch or release work:

```bash
git remote -v
git status --short
```

Never push to the `upstream` remote. It is read-only from our workflow's perspective; contributions reach it through GitHub pull requests.

## Product scope

Before implementing a feature, confirm all of the following:

- It solves a problem that can reasonably affect CloudCLI users outside Web.Eng.
- It does not hard-code Web.Eng names, infrastructure, defaults, credentials, or workflows.
- It can be explained as one focused upstream change.
- Existing upstream issues and pull requests have been searched for overlap.
- A non-trivial feature has been discussed in an upstream issue before substantial implementation, as required by `CONTRIBUTING.md`.
- Its ongoing conflict and maintenance cost is proportionate to its user value.

Bug fixes may proceed directly when the problem and expected behavior are clear.

Do not add organization-only features by default. If Web.Eng genuinely needs one, pause and get an explicit decision to accept permanent fork maintenance before implementation. Record that exception in an organization issue.

## Branch model

### `main`: exact upstream mirror

`main` represents `upstream/main` and is never a development branch.

- Do not commit directly to `main`.
- Update it only by fast-forwarding from `upstream/main`.
- Do not merge the Web.Eng integration branch or feature branches into it.
- Protect it on GitHub against direct pushes and force pushes.

Update it with:

```bash
git fetch upstream --prune
git switch main
git merge --ff-only upstream/main
git push origin main
```

If the fast-forward fails, stop and investigate. Do not create a merge commit to hide divergence.

### `webeng`: organization runtime and integration branch

`webeng` is the version Web.Eng builds and uses. It combines `main` with reviewed upstream-candidate features that have not yet landed in an upstream release.

- Treat `webeng` as a shared, protected branch.
- Once it exists, make it the GitHub default branch so organization pull requests target the version Web.Eng actually uses.
- Merge into it only through organization pull requests.
- Do not develop directly on it.
- Do not place commits on it that exist nowhere else; every functional change must have an owning feature branch and issue.
- Keep feature branches alive while their upstream pull requests are open.
- Merge updated `main` into `webeng` after upstream updates, then run the complete verification suite.
- Never force-push `webeng` during normal work.

Once an upstream pull request is merged, update `main` and merge it into `webeng`. The upstream implementation becomes authoritative. Resolve any difference in favor of upstream unless a documented compatibility reason requires a temporary adaptation.

If history becomes difficult after several upstream squash merges, rebuild `webeng` from current `main` plus the still-open feature branches in a temporary branch. Test it fully and replace the shared branch only after explicit team coordination.

### Feature branches: one upstream change each

Create every upstream-bound feature branch from the latest `main`, never from `webeng`:

```bash
git fetch upstream origin --prune
git switch main
git merge --ff-only upstream/main
git push origin main
git worktree add ../claudecodeui-worktrees/short-description \
  -b feat/short-description main
```

Use Conventional Commit branch and commit intent where practical:

- `feat/...` for user-facing capabilities
- `fix/...` for defects
- `refactor/...` for behavior-preserving structural work
- `docs/...`, `test/...`, or `chore/...` where appropriate

A feature branch must remain independently understandable and testable against upstream. Do not make it depend on unrelated Web.Eng changes. If stacking is unavoidable, document the dependency and wait for the base change to land before presenting the dependent work upstream.

## Issue and pull request flow

Use issues in `TheWebEng/claudecodeui` as the operational record. Link the upstream issue or pull request whenever one exists.

Recommended labels:

- `upstream-candidate`
- `upstream-discussion`
- `upstream-submitted`
- `upstream-feedback`
- `upstream-merged`
- `blocked`

For an upstream candidate:

1. Search upstream issues and pull requests.
2. Open or join an upstream discussion for a non-trivial feature.
3. Create the feature branch from `main`.
4. Implement and verify the feature in its own worktree.
5. Open an organization pull request from the feature branch into `webeng` so Web.Eng can review and use it.
6. Open an upstream pull request from the same feature branch into `siteboon/claudecodeui:main`.
7. Apply upstream review feedback on the feature branch.
8. If the branch was already merged into `webeng`, bring later review changes into `webeng` through a follow-up organization pull request.
9. Keep the branch until the upstream pull request is merged or deliberately closed.

Upstream pull requests must follow `CONTRIBUTING.md`: focused scope, conventional title, a clear why, linked issues, passing build, and screenshots or recordings for visual changes.

The Web.Eng workspace also requires every agent-created pull request to include a `Request context` section containing the original request, relevant clarifications, assumptions, and any meaningful difference between the request and implementation.

## When upstream is slow or declines a change

An open upstream pull request does not block Web.Eng from using a reviewed feature from `webeng`.

However, delay must not silently turn a temporary patch into permanent fork debt:

- Keep responding to upstream feedback promptly.
- Rebase or update the feature when upstream changes nearby code.
- Reassess open features during every upstream release sync.
- If upstream rejects the feature on product grounds, default to removing it from `webeng`.
- Keep a rejected feature only after an explicit team decision that its value exceeds its ongoing merge, testing, and support cost.

## Local checkout and worktrees

The checkout at `/Users/eugene/Documents/Code/webeng-code/claudecodeui` is the control checkout. Keep it on `main` for fetching, comparison, and worktree management.

Use persistent worktrees for active development instead of additional clones:

```text
/Users/eugene/Documents/Code/webeng-code/
├── claudecodeui/                 # control checkout on main
└── claudecodeui-worktrees/
    ├── webeng/                   # integrated version used by Web.Eng
    └── <feature-name>/            # one worktree per active feature
```

Example:

```bash
git -C /Users/eugene/Documents/Code/webeng-code/claudecodeui worktree add \
  /Users/eugene/Documents/Code/webeng-code/claudecodeui-worktrees/my-feature \
  -b feat/my-feature main
```

Use Codex-managed worktrees for bounded agent tasks, but do not globally install or operate the long-lived Web.Eng build from a temporary agent worktree.

Each worktree needs its own `node_modules` installation:

```bash
npm ci
```

Do not symlink `node_modules` between worktrees. Native modules and lifecycle scripts make that fragile.

When a branch is finished and safely merged or abandoned, remove its worktree and then delete the local branch. Never delete a worktree with uncommitted changes.

## Isolated development environments

Run development with:

```bash
npm run dev
```

Each simultaneously active worktree must have its own ignored `.env` with unique ports and database path:

```env
SERVER_PORT=3011
VITE_PORT=5181
HOST=0.0.0.0
DATABASE_PATH=/Users/eugene/.cloudcli-dev/my-feature/auth.db
```

Open the frontend at the configured `VITE_PORT`.

- Do not commit `.env`, databases, credentials, tokens, or session data.
- Do not point multiple running development servers at the same SQLite database.
- Use a database under `~/.cloudcli-dev/<worktree>/` for feature work.
- Stop CloudCLI before copying a SQLite database for representative testing.
- Back up `~/.cloudcli/auth.db` before testing migrations or switching materially different builds against live personal data.

## Verification requirements

Define the feature's expected behavior before editing. Verify it rather than assuming the implementation works.

Minimum checks for code changes:

```bash
npm run typecheck
npm run build
```

Also run:

- Focused tests for the changed behavior.
- ESLint for changed frontend and server files, or the complete lint command when appropriate.
- Existing integration tests covering affected database, provider, or WebSocket behavior.
- Browser testing against the running development instance for UI work.
- Both desktop and narrow mobile layouts for responsive changes.
- Copy, cancel, retry, failure, loading, and confirmation states where applicable.

Visual pull requests require before-and-after screenshots or a short recording. Keep verification artifacts out of the repository unless upstream explicitly expects them there; use GitHub-hosted attachments in the pull request description.

Before merging a feature into `webeng`, test the feature alone. After merging it, test the combined `webeng` branch because independently correct features can still conflict in the integrated application.

## Installing and using the Web.Eng build

Only install the organization build from the persistent `webeng` worktree after its checks pass:

```bash
npm ci
npm run build
npm install -g .
cloudcli status
```

Run `cloudcli` normally. Using the same hostname and port preserves the existing PWA origin.

Do not run `cloudcli update` while using the organization build; that command replaces it with the published upstream package. To intentionally return to the official release:

```bash
npm install -g @cloudcli-ai/cloudcli@latest
```

Tag known-good organization builds using a clear non-upstream tag such as `webeng-v<upstream-version>.<build>`. Do not change package versions on upstream feature branches merely to identify local builds.

## Keeping attribution clear

- Contributors must commit with an email address linked to their own GitHub account.
- Do not use shared human identities for commits or pull requests.
- Never add `Co-Authored-By` or automated signature lines to commits or pull requests.
- Each contributor should open or meaningfully own the pull request for their work where practical.
- The organization owns the repository; individuals retain authorship of their commits, reviews, issues, and upstream pull requests.

## Definition of done

An upstream-bound feature is done only when:

- Its general user value and scope are documented.
- Overlapping upstream work was checked.
- The feature branch was created from current `main`.
- Focused tests, type checks, linting, production build, and relevant browser flows pass.
- The organization pull request into `webeng` is reviewed.
- The upstream pull request is open with the required context and visual evidence.
- Upstream feedback has been addressed or clearly recorded.
- The integrated `webeng` build has been tested.
- The feature's status and maintenance decision are reflected in its organization issue.

The long-term success measure is not how many custom features the fork contains. It is how many useful improvements reach upstream while Web.Eng remains able to run and test them safely during review.
