# codex-delegate

Codex CLI stops in ways that look alike from outside: an `idle` pane can mean the work is finished, a question is waiting, or a usage limit was hit. The steps here make completion observable by requiring a report file, and the orchestrator re-runs the verification commands itself before accepting that report.

The skill covers starting Codex in a separate Herdr tab, handing it a task file, waiting without polling, resuming after a usage limit, sending review findings back to the same agent, and closing the tab afterwards.

Codex is started without options and commits its own changes. The skill assumes the Codex config disables approvals and the sandbox, so use it only where running Codex that way is acceptable.

## Requirements

- [Herdr](https://herdr.dev/), with the skill running inside a Herdr pane
- Codex CLI and `jq`
- `approval_policy = "never"` and `sandbox_mode = "danger-full-access"` in `~/.codex/config.toml`

## Related skills

These skills are not part of this plugin. The steps that hand off to them can be followed by hand when they are not installed.

- `pr-flow` prepares the worktree and creates the pull request. It lives outside this marketplace.
- `third-party-review` and `merged-branch-cleanup` are plugins in this marketplace.
