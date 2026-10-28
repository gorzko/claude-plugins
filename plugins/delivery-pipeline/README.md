# gorzko-plugins

Claude Code plugins. One for now: **delivery-pipeline**.

## delivery-pipeline

An autonomous delivery pipeline for a GitHub repository, run from Claude Code cloud sessions.

| Skill | What it does |
|---|---|
| `orchestrate-delivery` | Makes a session the repo's orchestrator, acting as project manager and tech lead. It keeps a ranked backlog in a "Delivery plan" issue and spawns worker sessions that implement issues as PRs. It drives each PR through claude[bot] reviews to the `ready-for-uat` label, and asks the owner only through `needs-owner` issues and push notifications. It never merges: the owner merges after their own testing. |
| `setup-claude-review` | Installs a hardened PR-review workflow. claude[bot] then submits a real APPROVED or CHANGES_REQUESTED review when a PR is marked "Ready for review", or on an `@claude review` comment. It also covers stacked PRs and troubleshooting. |

### Install

In Claude Code (CLI or desktop):

```
/plugin marketplace add gorzko/claude-plugins
/plugin install delivery-pipeline@gorzko-plugins
```

For cloud sessions of a repository (orchestrators and the workers they spawn), add this to the repository's `.claude/settings.json`:

```json
{
  "extraKnownMarketplaces": {
    "gorzko-plugins": { "source": { "source": "github", "repo": "gorzko/claude-plugins" } }
  },
  "enabledPlugins": { "delivery-pipeline@gorzko-plugins": true }
}
```

### Start an orchestrator

Open a cloud session on the repository, in auto permission mode, and send:

```
/delivery-pipeline:orchestrate-delivery
```

You can add Config overrides in the same message, for example `MAX_PARALLEL_WORKERS: 1`. The skill creates its own wake-up trigger. That trigger only says "load the skill and run your turn loop", so updating this plugin updates every orchestrator.

To add the review workflow to a repository yourself, run `/delivery-pipeline:setup-claude-review`.

## License

MIT
