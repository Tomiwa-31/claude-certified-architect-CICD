# Testing the Claude PR Review pipeline in GitHub Actions

`ci_pipeline.sh` in the repo root is a teaching snippet: it is a GitHub Actions
workflow written in YAML, not a shell script, and it will not run as-is
(placeholder `--jq '...'`, inline comments after `\` line-continuations, no
`claude` CLI install, and a missing `.claude/review_schema.json`).

The runnable version lives at `.github/workflows/claude-pr-review.yml`, with the
structured-output schema at `.claude/review_schema.json`.

## One-time setup (your own fork)

Actions only run in a repository you can push to, so work in a fork.

1. On GitHub, open the upstream repo and click **Fork** (keep the name).
2. Point your local clone at the fork:

   ```bash
   git remote add fork https://github.com/<your-user>/claude-certified-architect.git
   git push fork main
   ```

3. In the fork: **Settings → Secrets and variables → Actions → New repository secret**
   - Name: `ANTHROPIC_API_KEY`
   - Value: an API key from console.anthropic.com with credit on it.
4. In the fork: **Settings → Actions → General** → make sure Actions are enabled.

## Run it

**Option A, a real PR (the normal trigger)**

```bash
git checkout -b test/ci-review
# make any small code change, e.g. edit api_check.py
git commit -am "test: trigger Claude PR review"
git push fork test/ci-review
```

Open a PR on GitHub from `test/ci-review` into your fork's `main` (not upstream).
The **Claude PR Review** check appears on the PR; it fails if any finding has
`severity: critical`.

**Option B, manual run** — Actions tab → *Claude PR Review* → *Run workflow* →
enter an existing PR number.

## Reading the output

The *Run Claude review* step prints `findings.json`:

```json
{ "summary": "...", "findings": [ { "severity": "critical|high|medium|low",
  "file": "...", "line": 12, "title": "...", "description": "..." } ] }
```

## Notes

- Secrets are not exposed to PRs opened from other people's forks; open the PR
  inside your own fork.
- The workflow pipes the PR diff into `claude -p` on stdin and asks for
  structured output with `--json-schema`; the result is read from the
  `structured_output` field of the CLI's JSON envelope.
