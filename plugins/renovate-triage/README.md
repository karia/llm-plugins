# renovate-triage

Passing CI is not enough to accept a dependency update. This skill checks Renovate pull requests against the runtime the repository deploys to, handles necessary fixes, then approves and merges compatible pull requests one at a time. It also checks for newly created pull requests and pending updates in the Dependency Dashboard before cleaning up and reporting the results.

## Requirements

- GitHub CLI authenticated with permission to approve and merge pull requests

## Related skills

- `secret-scan-before-push` checks changes before pushing manual fixes to Renovate branches. It is a plugin in this marketplace.
