# Weakened Test Check for AI Agents

When a test fails, the quickest "fix", for a person or a coding agent, is
often to make the test check less. This GitHub Action reads every test a
pull request changes, compares it with the base branch, and keeps **one PR
comment** that says which changes deserve a human look:

| Needs review | Worth a look | Safe |
|---|---|---|
| Assertion weakened (checks less than before) | Wait or timeout added, `force: true` | Locator only |
| Expected value changed | Snapshot updated | Renamed |
| Test disabled or deleted | | |

It also says who made the change (a person, an AI-assisted commit, or a
coding agent such as Copilot, Cursor, Devin or Claude), based on commit
authors and trailers.

- **No account, nothing uploaded.** The review runs on your runner with
  git; your code never leaves it.
- **Never blocks by default.** Add `strict: true` to fail the check when a
  change needs review.
- Built for Playwright, and it also reads Jest/Vitest-style tests
  (`test(...)`/`it(...)` with `expect(...)`) in `*.spec.*`/`*.test.*`
  files (JS and TS).

## Setup

```yaml
name: Test change review
on: pull_request

jobs:
  review:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write
    steps:
      - uses: actions/checkout@v7
        with:
          fetch-depth: 0   # needs the base branch history
      - uses: actions/setup-node@v7
        with:
          node-version: 22
      - uses: sekharsdet/test-change-review@v1
```

That's all: no secrets, no config.

## Inputs

| Input | Default | What it does |
|---|---|---|
| `strict` | `false` | Fail the step when a change needs review |
| `comment` | `true` | Post and update one PR comment. `false` prints the review in the log only |
| `base` | the PR's base branch | Git ref to compare against |
| `version` | `0.6` | turnsignal npm version or range (pin an exact version to approve upgrades yourself) |
| `github-token` | `github.token` | Posts the comment |

## Security

- The token is used only to post the comment. Packages are installed in a
  separate step with no token, with install scripts disabled, and without
  optional peer packages. The review itself runs from a single bundled
  file with no runtime dependencies.
- Test titles and code from the pull request are printed with GitHub
  workflow commands suspended, so they can't be read as `::commands::`.
- On pull requests from forks the token is read-only: the review still
  runs and prints to the log, but GitHub doesn't allow the comment.

## How it works

This Action runs `turnsignal governance` from the open-source
[`turnsignal`](https://github.com/sekharsdet/turnsignal) package. It parses
each changed test before and after the change, so it compares structure
(the assertions and steps) rather than text. That's why a renamed variable
or a moved locator is marked safe, while `toHaveText('Paid')` becoming
`toBeVisible()` is marked as weakened.

If you also want to know which failing tests are new in a pull request,
which were already failing on main, and which are flaky, that's what
[TurnSignal](https://turnsignal.ai/?ref=test-change-review) does across CI
runs. It's a separate, optional service from the same author, and this
Action doesn't need it.

## License

[MIT](./LICENSE)
