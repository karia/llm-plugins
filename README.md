# llm-plugins

A personal plugin marketplace that works with both Claude Code and Codex CLI.

Each plugin ships a single skill. The skills themselves are written in Japanese, because that is the language I work in.

## Plugins

| Plugin | Description |
| --- | --- |
| `secret-scan-before-push` | Check repository visibility and scan the commits you are about to push for credentials and internal identifiers. |
| `merged-branch-cleanup` | Return to the default branch and safely delete the local branch and worktree once a pull request is merged. |

## Install

### Claude Code

```
/plugin marketplace add karia/llm-plugins
/plugin install <plugin>@llm-plugins
```

### Codex CLI

```
codex plugin marketplace add karia/llm-plugins
codex plugin add <plugin>@llm-plugins
```

## Layout

```
.
├── .claude-plugin/marketplace.json   # marketplace catalog read by Claude Code
├── .agents/plugins/marketplace.json  # marketplace catalog read by Codex CLI
└── plugins/
    └── <plugin>/
        ├── .claude-plugin/plugin.json
        ├── .codex-plugin/plugin.json
        └── skills/<skill>/SKILL.md
```

The two catalogs describe the same set of plugins in the format each host expects, and the two manifests inside a plugin point at the same `skills/` directory. Adding a plugin means adding an entry to both catalogs.

Codex needs `"skills": "./skills/"` in its manifest; Claude Code discovers `skills/` by convention. Hook scripts, if any are added later, cannot rely on `CLAUDE_PROJECT_DIR` under Codex — use `CLAUDE_PLUGIN_ROOT` / `PLUGIN_ROOT` instead, and keep hook output from starting with `[` so Codex does not read it as JSON.

## License

MIT
