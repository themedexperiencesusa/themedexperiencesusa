# Codebase Audit Worksheet (2026-10-02)

Scope: whole repo (README, .gitignore, 3 GitHub workflows). No application code exists.
Note: no "intake" or "ecosystem" agents are defined in this repo; this worksheet is written to be handed to whichever workers you assign.

| # | Sev | Area | Issue | Delegate to / Action | Status |
|---|-----|------|-------|----------------------|--------|
| 1 | HIGH | `.github/workflows/summary.yml` | **Script injection.** `${{ steps.inference.outputs.response }}` is expanded inline into a shell command inside single quotes. The response is model output derived from untrusted issue title/body, so a crafted issue can break out of the quotes and run commands with an `issues: write` token. | Security worker: use the already-defined `$RESPONSE` env var (`--body "$RESPONSE"`) and drop the inline expression. | Open |
| 2 | MED | `summary.yml` | Prompt injection: issue title/body is fed straight to the model and the output is posted publicly. | Security worker: delimit/quote the input, limit the output, consider restricting to maintainers or labelling. | Open |
| 3 | MED | `.github/workflows/label.yml` | `actions/labeler` needs `.github/labeler.yml`, which does not exist, so every PR run fails. Uses outdated `@v4` with `pull_request_target`. | CI worker: add labeler config (or delete the workflow); bump to a current major. Do not check out PR head code under `pull_request_target`. | Open |
| 4 | LOW | `.github/workflows/greetings.yml` | Placeholder messages ("Message that will be displayed on users' first issue") would post publicly. `actions/first-interaction@v1` is old; runs on `pull_request_target`. | Content worker: write real messages or remove; update action version. | Open |
| 5 | LOW | All workflows | Third-party actions pinned by tag, not commit SHA; no Dependabot for actions; no top-level `permissions:` default. | CI worker: pin to SHAs, add `.github/dependabot.yml` (github-actions ecosystem). | Open |
| 6 | LOW | `.gitignore` | WordPress template in a GitHub profile repo (no WordPress code). Ignores `/index.php`, `/wp-*.php`, `/readme.html`, `/license.txt`, which could silently hide future files. No trailing newline. | Repo-hygiene worker: replace with a minimal ignore (or confirm WordPress is planned). | Open |
| 7 | LOW | `README.md` | Placeholder text and the unedited template comment block; no real content or contact info. | Content worker: write profile content, remove template. | Open |
| 8 | INFO | Repo | No LICENSE, SECURITY.md, or CODEOWNERS. | Repo-hygiene worker: add as appropriate. | Open |

## Suggested order
1 -> 2 -> 3 -> 5 -> 4/6/7/8.

## Fix for #1 (reference)
```yaml
      - name: Comment with AI summary
        run: gh issue comment "$ISSUE_NUMBER" --body "$RESPONSE"
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          ISSUE_NUMBER: ${{ github.event.issue.number }}
          RESPONSE: ${{ steps.inference.outputs.response }}
```
