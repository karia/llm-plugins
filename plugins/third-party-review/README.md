# third-party-review

A review carried out in the session that wrote the code is not a review. It inherits every assumption the implementation was built on, which is exactly what a reviewer is there to question. So the reviewer starts from nothing: it gets the repository name and the pull request number, and reads the diff and the description the way anyone else would.

By default the reviewer is a subagent of the requesting session. An external coding agent is launched only when the requester asks for one.

Findings come back in a result file rather than as GitHub comments. A review posted straight to a pull request leaves comments on someone else's work and sends notifications before the author has read a word of it. The result file is written into an Obsidian note where one is available, and under `~/Documents/` otherwise.

The pull request stays in draft from the start of the review until the end. Taking it out of draft is the author's call, and belongs to the `pr-flow` skill.
