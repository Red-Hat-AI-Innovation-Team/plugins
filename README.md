# Red Hat AI Innovation Team — Coding Agent Plugins

Plugin marketplace for the [Red Hat AI Innovation Team](https://ai-innovation.team).

## Available Plugins

| Plugin | Description | Status |
|--------|-------------|--------|
| [its-hub](https://github.com/Red-Hat-AI-Innovation-Team/its_hub) | Inference-time scaling for LLMs — voting, scoring, and search | Ready |
| [sdg-hub](https://github.com/Red-Hat-AI-Innovation-Team/sdg_hub) | Synthetic data generation for LLM training and evaluation | Coming soon |
| [training-hub](https://github.com/Red-Hat-AI-Innovation-Team/training_hub) | LLM training, continual learning, and reinforcement learning | Coming soon |

## Install

### Claude Code

```
/plugin marketplace add Red-Hat-AI-Innovation-Team/plugins
```

Then browse the **Discover** tab to see available plugins, or install directly:

```
/plugin install its-hub@Red-Hat-AI-Innovation-Team/plugins
```

### Other Agents

Each plugin repo contains its own install instructions for Gemini CLI, Codex CLI, Cursor, and OpenCode. See the individual repos above.

## Adding a Plugin

To add a new plugin to this marketplace:

1. Create a `.claude-plugin/plugin.json` manifest in the plugin repo
2. Add an entry to `.claude-plugin/marketplace.json` in this repo
3. The `source` field should be the GitHub shorthand: `Red-Hat-AI-Innovation-Team/<repo-name>`
