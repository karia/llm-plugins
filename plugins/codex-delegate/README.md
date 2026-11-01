# codex-delegate

Codex CLI stops in ways that look alike from outside: an `idle` pane can mean the work is finished, a question is waiting, or a usage limit was hit. The steps here make completion observable by requiring a report file, and confirm it by re-running the checks rather than trusting the summary.

The skill covers starting Codex in a sibling Herdr pane, handing it a task file, waiting without polling, resuming after a usage limit, sending review findings back to the same agent, and closing the pane afterwards.

It requires running inside [Herdr](https://herdr.dev/) and pairs with the `pr-flow` skill, which prepares the worktree Codex works in and creates the pull request once the work is verified.
