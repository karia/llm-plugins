# third-party-review

A review carried out in the session that wrote the code is not a review. It inherits every assumption the implementation was built on, which is exactly what a reviewer is there to question. So the reviewer gets none of the implementation history: it gets the repository name, the pull request number, and the bar to judge against (what kind of artifact it is, who reads it, and what level it must meet), and reads the diff and the description the way anyone else would.

The reviewer gives a verdict first and lists only problems that should block the merge, at most five, each with how it breaks in normal use and whether it was actually verified. The requesting session does not take findings at face value: it recommends fixing only what it can confirm and what is worth the added text, and a follow-up review checks only the previous problems and regressions.

By default the reviewer is a subagent of the requesting session. An external coding agent is launched only when the requester asks for one.

Findings come back in a result file rather than as GitHub comments. A review posted straight to a pull request leaves comments on someone else's work and sends notifications before the author has read a word of it. The result file is written into an Obsidian note where one is available, and under `~/Documents/` otherwise.

The pull request stays in draft from the start of the review until the end. Whether to take it out of draft after that is decided by the `pr-flow` skill.
