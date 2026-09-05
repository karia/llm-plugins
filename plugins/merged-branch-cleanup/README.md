# merged-branch-cleanup

Deleting a branch or a worktree cannot be undone, so the steps here verify before they destroy: the pull request is confirmed merged, the working tree is confirmed clean, and the default branch name is read from the remote rather than assumed to be `main`.

The skill covers returning to the default branch and updating it, removing the merged local branch and its worktree, rebasing onto the fresh default branch, and updating a pull request description once the work it depended on has landed.

It pairs with the `pr-flow` skill, which does its work inside a worktree under `.claude/worktrees/` — this is the half that takes that worktree away again.
