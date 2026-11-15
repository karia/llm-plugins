# pr-self-review

Takes a pull request you wrote through three reviews, fixes what needs fixing, and repeats until a full round produces no fix. It runs to completion without asking questions.

1. An inspection for leftovers of the work that produced the pull request: references to the request, change history in comments, debug output.
2. A review by an agent that never saw the implementation.
3. A prompt audit through the `claude-api` skill, only when the diff contains prompts written for an LLM.

Each finding is judged on its own. One fix is one commit, and nothing is pushed until the loop ends; the fix commits are then tidied with a rebase and pushed once. Findings that point at a specification problem, or that belong in a separate pull request, are left untouched and reported at the end together with the findings judged not to need a fix.

The skill is self-contained. The inspection criteria, the third-party review procedure, and the writing norms live under `skills/pr-self-review/references/`. The last two are copies of the `third-party-review` and `tech-writing-style` plugins in this marketplace, adapted to run without asking the requester. The inspection criteria are written for this skill; the idea comes from the `sanitize-artifacts` skill in [kotek-7/dotfiles](https://github.com/kotek-7/dotfiles).
