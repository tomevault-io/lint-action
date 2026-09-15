# TomeVault lint action

Gate your AI instruction files in CI. This action runs [`tomevault lint`](https://www.npmjs.com/package/tomevault) over every instruction file in your repo (`CLAUDE.md`, `AGENTS.md`, `GEMINI.md`, `.cursor/rules/*`, `.windsurf/rules/*`, `copilot-instructions.md`, `.claude/skills/*/SKILL.md`), checks each for safety, clarity, and whether it will actually load, and fails the build if anything blocks.

It runs entirely on your runner. No token, no network call to us, nothing leaves the box, so it is safe on private repos. The one exception is the optional pull-request comment below, which needs a token to talk to the GitHub API. Even then, no file content leaves your repository.

## Quick start

```yaml
name: TomeVault lint
on:
  push:
  pull_request:

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
      - uses: tomevault-io/lint-action@v1
```

On every push and pull request the action scans your instruction files, drops inline annotations on the offending lines in the **Files changed** tab, writes a job-summary table, and sets the build status. By default warnings are reported but do not fail the build, so you can adopt the gate without it red-lining day one.

## Make it a required check

The action reports a status, but a status only **blocks a merge** once you mark it required in branch protection:

1. Open **Settings → Branches → Branch protection rules** for your default branch.
2. Enable **Require status checks to pass before merging**.
3. Add **TomeVault lint** to the required checks.

After that, a PR that introduces a load-blocker or a safety fail cannot merge until it is fixed.

## Inputs

| Input | Default | Description |
| --- | --- | --- |
| `paths` | _(whole repo)_ | Space- or newline-separated files/dirs to lint. Omit to auto-discover every instruction file. |
| `strict` | `false` | Treat warnings as errors, so any warning fails the build. |
| `max-warnings` | _(no limit)_ | Fail once warnings exceed this count. Ignored when `strict` is true. |
| `version` | `1` | CLI version to run. `1` tracks the latest 1.x. Pin an exact version (e.g. `1.8.0`) for a gate that never shifts under you. |
| `working-directory` | `.` | Directory to run in. |
| `comment` | `false` | Post the verdict as one pull-request comment that updates itself on every run. Needs `github-token` and CLI 1.8.6 or newer. Ignored outside a pull request. |
| `github-token` | _(none)_ | Token used to post that comment, needing `pull-requests: write`. Only read when `comment` is true. |

```yaml
      - uses: tomevault-io/lint-action@v1
        with:
          strict: true
          paths: |
            CLAUDE.md
            .cursor/rules
          version: 1.8.0
```

## Outputs

| Output | Description |
| --- | --- |
| `passed` | `true` if the gate passed, `false` otherwise. |
| `errors` | Count of error-severity findings (safety fails and load-blockers). |
| `warnings` | Count of warning-severity findings. |
| `infos` | Count of info-severity advisories. |

To branch on the result instead of failing the build, let the step continue and read its outputs:

```yaml
      - uses: tomevault-io/lint-action@v1
        id: tomevault
        continue-on-error: true
      - if: steps.tomevault.outputs.passed == 'false'
        run: echo "found ${{ steps.tomevault.outputs.errors }} errors"
```

## Put the verdict in the pull request

By default the action drops inline annotations on the offending lines and writes a job-summary table. The summary lives on the workflow-run page, which a reviewer only sees if they click into the run. Turn on `comment` and the verdict is posted into the pull-request conversation instead, where the review is already happening.

```yaml
jobs:
  lint:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write
    steps:
      - uses: actions/checkout@v5
      - uses: tomevault-io/lint-action@v1
        with:
          comment: true
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

One comment per pull request, rewritten in place on every push, so a long-running branch does not collect a stack of them. A clean run is one line. Files are itemised only when they carry an error or a warning, and the findings fold lists the rule and the line, never the content of the file it read.

Three things worth knowing:

- The comment is posted whether the gate passes or fails, because a failing gate is when it is most useful.
- A pull request from a fork carries a read-only token, so the post is refused by GitHub. The action logs a warning and the gate result is unaffected.
- It needs `tomevault` 1.8.6 or newer. If you have pinned an older `version`, the action says so in a warning and runs the gate without the comment rather than failing it.

## Action or service

This action is the CI gate you run inside your own pipeline. It blocks a merge once you mark it required, and by default needs no token and sends nothing off your runner.

[TomeVault](https://tomevault.io/) is the hosted service for teams that would rather have their instruction files watched for them: continuous monitoring across every repo, with a dated record of what changed and when. Use the action for a self-hosted gate, the service for continuous cover, or both.

## Versioning

The moving `v1` tag tracks the latest 1.x of this action. Pin `@v1` to get non-breaking updates automatically, or pin an exact release tag for a gate that never changes. The action's bundled CLI defaults to `tomevault@1`; override it with the `version` input.

## License

MIT
