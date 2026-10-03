# Codebase Issues Worksheet (2026-10-03)

Scope: whole repo (5 tracked files: 3 workflows, README, .gitignore).
Note: no "intake agent" or "ecosystem agent workers" are defined in this repo or session; this worksheet was written directly and is ready to hand to whichever workers you designate.

| # | Severity | File | Problem | Recommended action | Owner | Status |
|---|----------|------|---------|--------------------|-------|--------|
| 1 | HIGH (security) | .github/workflows/summary.yml | `${{ steps.inference.outputs.response }}` is interpolated directly into a single-quoted shell command. The response is model output derived from untrusted issue title/body, so a crafted issue can break out of the quotes and run commands with `issues: write` + `GH_TOKEN`. The `RESPONSE` env var is set but never used. | Use `--body "$RESPONSE"` (env var only, no `${{ }}` in `run`). Also pass issue title/body via env, not inline in `prompt`. | Workflow/security worker | TODO |
| 2 | MED | .github/workflows/label.yml | `actions/labeler` needs `.github/labeler.yml`, which does not exist, so the job fails on every PR. Uses `pull_request_target` and outdated `@v4`. | Add `.github/labeler.yml` (or remove the workflow); bump to current major; keep no checkout of PR code. | Workflow worker | TODO |
| 3 | LOW | .github/workflows/greetings.yml | Placeholder messages ("Message that will be displayed..."); `first-interaction@v1` is outdated; runs on `pull_request_target`. | Write real messages, bump version. | Workflow worker | TODO |
| 4 | LOW (supply chain) | all workflows | Third-party actions pinned to mutable tags, not commit SHAs. | Pin to SHAs, add Dependabot for `github-actions`. | Security worker | TODO |
| 5 | LOW | summary.yml | Fires for any issue author, no rate/permission guard; `models: read` dependent on feature availability for this account. | Confirm GitHub Models access or disable; consider author-association guard. | Owner decision | TODO |
| 6 | INFO | README.md | Profile README is a placeholder with the template comment left in. | Fill in profile content. | Content worker | TODO |
| 7 | INFO | repo | No tests/CI beyond issue/PR automation; no Dependabot, CODEOWNERS or SECURITY.md. | Add if desired. | Owner decision | TODO |
