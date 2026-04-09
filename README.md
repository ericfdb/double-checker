# Double Checker

AI code review on every PR. Powered by [OpenAI Codex](https://openai.com/index/introducing-codex/).

## Usage

Add to `.github/workflows/double-check.yml`:

```yaml
name: Double Check
on: pull_request

permissions:
  contents: read
  pull-requests: write

concurrency:
  group: double-check-${{ github.event.pull_request.number }}
  cancel-in-progress: true

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - uses: ericfdb/double-checker@main
        with:
          openai-api-key: ${{ secrets.OPENAI_API_KEY }}
```

Then add your OpenAI API key as a repository secret (`OPENAI_API_KEY`).

## How it works

1. A PR is opened or updated
2. Any in-progress review for the same PR is cancelled automatically (concurrency group)
3. The action reviews the full diff (`base..HEAD`) using Codex
4. A PR review is submitted — **approve** if clean, **request changes** if `[P0]` issues are found
5. The status check passes or fails accordingly

## Review format

Codex uses its built-in review rubric with severity tags:

- **`[P0]`** — Critical: blocks the PR (bugs, security, broken builds)
- **`[P1]`** — Important: should be addressed (regressions, missing edge cases)
- **`[P2]`** — Moderate: worth noting (maintainability, unclear logic)
- **`[P3]`** — Minor: optional improvements

By default, only `[P0]` findings trigger a "request changes" review and fail the check. Use the `severity` input to widen the threshold (e.g., `P1` rejects on both `[P0]` and `[P1]`).

## Options

| Input | Default | Description |
|---|---|---|
| `openai-api-key` | `''` | OpenAI API key (leave empty if auth is pre-configured) |
| `model` | `gpt-5.4` | Model for review |
| `severity` | `P0` | Minimum severity to reject: `P0`, `P1`, `P2`, or `P3` |
| `fail-on-reject` | `true` | Fail the status check if issues at or above the severity threshold are found |

## Required permissions

```yaml
permissions:
  contents: read
  pull-requests: write
```

## Branch protection

To enforce the gate, enable branch protection on your default branch and require the `review` status check to pass. You can also require pull request reviews, which will block merge on any "changes requested" verdict. The check name comes from the job name in your workflow (default: `review`).

## License

MIT
