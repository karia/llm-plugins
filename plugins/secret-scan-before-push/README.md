# secret-scan-before-push

Pushing to a public repository exposes every commit in the range, including the diff of a pull request that has not been merged yet. Once it is out, you cannot undo the fact that GitHub and crawlers may have taken a copy, so the check has to happen before the push.

The skill walks through:

1. Confirming the visibility of the target repository.
2. Determining which commits this push actually publishes.
3. Scanning that range twice — an automated secret scan for known credential formats, and a manual read of the diff for context-dependent leaks such as cloud resource identifiers, internal host names, and non-public URLs.
4. Stopping and asking before publishing or rewriting anything questionable.

It also covers what to do when the push has already happened.

`gitleaks` is used for the automated pass when available, and run through `mise x gitleaks@latest` otherwise. Neither is required for the skill to be useful; the manual review is the half that finds what tools cannot.
