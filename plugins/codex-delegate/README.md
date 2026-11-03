# codex-delegate

Codex CLI stops in ways that look alike from outside: an `idle` pane can mean the work is finished, a question is waiting, or a usage limit was hit. The steps here make completion observable by requiring a report file, and the orchestrator re-runs the verification commands itself before accepting that report.

The skill covers starting Codex in a sibling Herdr pane, handing it a task file, waiting without polling, resuming after a usage limit, sending review findings back to the same agent, and closing the pane afterwards.

Codex starts in `workspace-write` with the orchestrator committing its changes. `--yolo` removes both approvals and the sandbox, so it is used only when explicitly requested.

## Requirements

- [Herdr](https://herdr.dev/), with the skill running inside a Herdr pane
- Codex CLI and `jq`

## Related skills

These skills are not part of this plugin. The steps that hand off to them can be followed by hand when they are not installed.

- `pr-flow` prepares the worktree and creates the pull request. It lives outside this marketplace.
- `third-party-review` and `merged-branch-cleanup` are plugins in this marketplace.
