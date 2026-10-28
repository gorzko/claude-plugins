---
name: setup-claude-review
description: Install or repair the formal Claude PR-review workflow in a GitHub repository. The workflow lets claude[bot] submit real APPROVED / CHANGES_REQUESTED reviews when a PR is marked "Ready for review" or someone comments "@claude review". Use when a repo has no review workflow, has the stock claude-code-review.yml (it reviews every push and never approves), when reviews fail or are skipped, or when stacked PRs need reviews.
---

# Set up the Claude PR-review workflow

The result is a pair of workflow files that the owner merges through a PR. After that:
- Marking a PR "Ready for review", or opening a non-draft PR, starts one review. Pushes don't start reviews, so there are no duplicates.
- A PR comment containing `@claude review` from someone with write access starts one re-review.
- The review is submitted by `claude[bot]` (the Claude GitHub App). It's a different identity from the owner's account, so GitHub accepts APPROVE and REQUEST_CHANGES even on PRs that the owner's own sessions opened. The body always starts with `Verdict:`, `Reviewed head: <sha>` and `Checks:` lines, which orchestrators parse.
- Findings are inline comments prefixed `[BLOCKING]` or `[NIT]`.

The templates are in `templates/`, next to this file:
- `claude-code-review.yml`: the reviewer.
- `claude.yml`: the general `@claude` assistant, which ignores `@claude review`.

## Security model (keep it intact when adapting)
- **`gate` job:** refuses fork PRs. Comment triggers count only from OWNER, MEMBER or COLLABORATOR.
- **`checks` job:** the only place PR code runs. It gets `contents: read`, no secrets, no OIDC, and `persist-credentials: false`. It hands over only a 400-line log artifact.
- **`review` job:** never executes PR code.
  - Claude runs in a checkout of the default branch, so CLAUDE.md, `.claude/` settings and hooks are trusted. The PR head is checked out read-only into `./pr`.
  - Context comes only from trusted authors.
  - Claude's tools are limited to Read/Glob/Grep, inline comments, and `gh pr review <this PR> --repo <this repo>`.
- **Concurrency:** set per job and per PR, so a later trigger cancels only the same PR's run.
- **Pinned to the reviewed commit:** the diff is computed locally for the head SHA under review, linked-issue text is included only from trusted authors, and an approval is dismissed automatically if the PR head moved while the review ran.
- **`claude.yml`:** only the owner, members and collaborators can summon `@claude` (author-association gate), and issue triggers fire on `opened` only.

Never add write tools, `Bash(*)`, or secrets to the `checks` job.

## Steps

### 1. Inspect the repository
- Find the default branch and check `.github/workflows/` for `claude-code-review.yml` and `claude.yml`.
  - If `claude-code-review.yml` exists and already has a `gate` job and the `Verdict:` prompt, it's this workflow. Skip to step 5 (stacks) or the troubleshooting section.
  - Otherwise it's absent or the stock version. Replace it.
- Work out the fast verification commands a contributor runs locally: lint plus unit tests, without building or E2E. Look in CLAUDE.md / AGENTS.md / CONTRIBUTING, package.json scripts, Makefile, pyproject and the existing CI. Examples:
  - **Node:** `INSTALL_CMD: npm ci --ignore-scripts` (or `npm install --ignore-scripts` when there's no lockfile). `CHECK_CMD: npm run lint && npm test` (or the repo's single script that does both).
  - **Python:** `INSTALL_CMD: pip install -e .[dev]`. `CHECK_CMD: ruff check . && pytest -q`.
  - **Static site with no tests:** `INSTALL_CMD: true`. `CHECK_CMD: true`. The reviewer then states that no checks exist.
  - Prefer `--ignore-scripts` (or the ecosystem's equivalent): install scripts of PR-added dependencies shouldn't run even in the sandboxed job.
- Check whether the runner needs a toolchain, such as a specific Node or Python version (`.nvmrc`, `engines`, `.python-version`).

### 2. Fill in the templates
- Copy both templates to `.github/workflows/` in the repo.
- In `claude-code-review.yml`:
  - Replace `__INSTALL_CMD__` and `__CHECK_CMD__`.
  - Replace the `# __TOOLCHAIN_SETUP__` comment line with a setup step, or delete the line. For example:
    ```yaml
          - uses: actions/setup-node@v4
            with:
              node-version-file: .nvmrc
    ```
  - Each command runs through `bash -c`, so `&&`, pipes and quotes work, e.g. `npm run lint && npm test -- --run`.
- Keep the review prompt generic. Project rules come from the repo's CLAUDE.md at review time, so don't copy them into the workflow.
- If `claude.yml` already exists, keep its customisations. Just make sure every `@claude` condition also has `!contains(<body>, '@claude review')`, so that the two workflows never both answer one comment.
- Leave no `__` placeholders. Check that the YAML parses, for example with `python3 -c "import yaml,sys;yaml.safe_load(open(sys.argv[1]))" <file>`.

### 3. Open a PR (never push workflows to the default branch directly)
- Branch from the default branch, commit only these two files, push, and open a non-draft PR titled "Add formal Claude PR-review workflow".
- The PR body explains what changes (from the bullets at the top of this file) and lists the owner's one-time setup below.
- Owner's one-time setup (only the owner can do these; never try to do them yourself):
  1. Install the Claude GitHub App on the repository: https://github.com/apps/claude, or run `/install-github-app` in Claude Code.
  2. Add the repository secret `CLAUDE_CODE_OAUTH_TOKEN` (from `claude setup-token`) or switch the action to `anthropic_api_key` with `ANTHROPIC_API_KEY`. The `/install-github-app` flow does both.
  3. Optional: add the repository variable `CLAUDE_REVIEW_MODEL` to override the default reviewer model.
  4. "Allow GitHub Actions to create and approve pull requests" is NOT needed, because the review is submitted with the app token.
- Don't expect the PR itself to get a review. The action refuses to run a workflow that differs from the default branch's copy ("Workflow validation failed"). That's expected, not a bug.

### 4. Verify after the owner merges
- On any open non-draft PR, the owner or a session posts one comment `@claude review`. Post it once only and never duplicate it. Don't let a GitHub autocomplete turn it into a mention of another bot.
- Expect a `Claude Code Review` run with the jobs gate → checks → review, then a review by `claude[bot]` whose state is APPROVED or CHANGES_REQUESTED and whose body starts with `Verdict:`.
- If it fails, see the troubleshooting section below.

### 5. Stacked PRs
- `pull_request` events use the workflow from the PR's own branch, and the action runs only if that copy is byte-identical to the default branch's.
- So whenever work is stacked on unmerged branches, copy both files from the default branch onto the top branch of the stack:
  - One commit containing only those two files.
  - No other changes.
  - Compare the blob SHAs afterwards.
- Every PR stacked above that point then gets reviews from "Ready for review".
- PRs below it, or any PR whose branch has an older copy, still get reviews through `@claude review`. Comment triggers always use the default branch's workflow.
- Whenever either file changes on the default branch, re-sync the stack tip.

## Troubleshooting
| Symptom | Cause / fix |
|---|---|
| Review job "skipped", or the Claude step says "Workflow validation failed" | The PR branch's copy of the workflow differs from the default branch's. Sync it (step 5), or use `@claude review`. |
| `Resource not accessible by integration` in the gate or context step | A `gh pr view` field needs more scope (e.g. `--comments` needs statusCheckRollup). Use `gh api repos/.../issues/N/comments` instead, as the template does. |
| checks job: `Repository not found` | The job lost `contents: read`; a private repo needs it to clone. |
| Claude result `is_error: true`, `permission_denials_count: 0`, no verdict review | Almost always the subscription's usage limit (the OAuth token shares quota with the owner's Claude sessions) or an API error. Re-run the failed job once after the limit resets. If it recurs on large PRs, raise `--max-turns` or the job timeout. |
| Review posted as COMMENTED instead of APPROVED | The action used the owner's token, not the app's. Check that the Claude GitHub App is installed on the repo. |
| Two answers to one `@claude review` | `claude.yml` lacks the `!contains(..., '@claude review')` exclusion. |
