# Red Hat AI Innovation Team — Plugin Marketplace

## Plugin Architecture

Every plugin in this marketplace follows a consistent 4-layer structure. This is the reference for contributors adding new plugins or maintaining existing ones.

### Layer 1: Agent Discovery Manifests

Each plugin supports 4 coding agents. Manifests are the only agent-specific files — everything else is shared.

```
<plugin-repo>/
├── .claude-plugin/
│   ├── plugin.json              # Claude Code discovery
│   └── marketplace.json         # Standalone marketplace fallback
├── .cursor-plugin/
│   └── plugin.json              # Cursor discovery
├── .codex-plugin/
│   ├── plugin.json              # Codex discovery (+ skills path)
│   └── INSTALL.md               # Clone + symlink instructions
├── .opencode-plugin/
│   ├── plugins/<name>.js        # JS plugin module (skill registration + bootstrap)
│   └── INSTALL.md               # opencode.json plugin config
```

**Naming conventions:**
- `plugin.json` name field: kebab-case (`its-hub`, `training-hub`, `sdg-hub`)
- OpenCode JS export: PascalCase (`ItsHubPlugin`, `TrainingHubPlugin`, `SDGHubPlugin`)

### Layer 2: Commands (Slash Commands)

Markdown files in `commands/` — each becomes a `/slash-command` in Claude Code and Cursor.

```
commands/
├── <prefix>-setup.md            # Guided first-run configuration
├── <prefix>-<primary>.md        # Primary domain action
└── <prefix>-<utility>.md        # Secondary utility action
```

**Frontmatter format:**
```yaml
---
description: "Short description"
argument-hint: "<args> [--flags]"
allowed-tools: ["Bash(${CLAUDE_PLUGIN_ROOT}/scripts/<script>.sh:*)"]
---
```

**Per-plugin commands:**

| Plugin | Setup | Primary | Utility |
|--------|-------|---------|---------|
| its-hub | `/its-setup` | `/its-scale` (scale a prompt) | `/its-server` (manage IaaS server) |
| training-hub | `/th-setup` | `/th-train` (run training) | `/th-estimate` (VRAM estimation) |
| sdg-hub | `/sdg-setup` | `/sdg-generate` (run a flow) | `/sdg-flows` (list/search/inspect) |

its-hub also has `/its-scale-batch` for batch processing.

### Layer 3: Skills (Contextual Routing)

Skills detect user intent and route to the appropriate command or script. Each plugin has exactly 2 skills.

```
skills/
├── setup-guide/
│   └── SKILL.md                 # First-time setup, routes to /<prefix>-setup
└── <domain>/
    └── SKILL.md                 # Domain intent detection, routes to commands
```

**Per-plugin skills:**

| Plugin | Setup Skill | Domain Skill |
|--------|-------------|--------------|
| its-hub | `setup-guide` | `inference-scaling` |
| training-hub | `setup-guide` | `training-guide` |
| sdg-hub | `setup-guide` | `data-generation` |

**SKILL.md frontmatter:**
```yaml
---
name: <skill-name>
description: "Trigger condition for this skill"
---
```

### Layer 4: Scripts (Shell Execution)

Shell scripts in `scripts/` — the actual executable logic. All scripts source `_env.sh` for Python venv resolution.

```
scripts/
├── _env.sh                      # Shared: resolve $PYTHON from venv
├── <prefix>_detect.sh           # Detect: library, installer, config, [gpu/server]
├── <prefix>_<primary>.sh        # Execute: primary domain action
└── <prefix>_<utility>.sh        # Execute: secondary utility action
```

**Detection script output format** (key=value, one per line):
```
library=installed|missing
installer=uv|pip|none
config=found|missing
server=running|stopped          # its-hub only
gpu=available|unavailable       # training-hub only
gpus=N                          # training-hub only
```

**Per-plugin scripts:**

| Plugin | Detect | Execute | Utility |
|--------|--------|---------|---------|
| its-hub | `its_detect.sh` | `its_scale.sh` | `its_server.sh` |
| training-hub | `th_detect.sh` | `th_train.sh` | `th_estimate.sh` |
| sdg-hub | `sdg_detect.sh` | `sdg_generate.sh` | `sdg_flows.sh` |

### Config Directories

Each plugin stores user-specific configuration in a dot-directory at the project root:

| Plugin | Config Dir | Config File | Env Override |
|--------|-----------|-------------|--------------|
| its-hub | `.its-hub/` | `config.json` | `ITS_HUB_CONFIG` |
| training-hub | `.training-hub/` | `config.json` | `TRAINING_HUB_CONFIG` |
| sdg-hub | `.sdg-hub/` | `config.json` | `SDG_HUB_CONFIG` |

All config directories are gitignored.

## Data Flow

```
User intent (natural language)
    │
    ▼
Skill detects intent (SKILL.md)
    │
    ▼
Skill runs detection script (<prefix>_detect.sh)
    │
    ▼
Skill routes to command (/prefix-action)
    │
    ▼
Command runs execution script (<prefix>_action.sh)
    │
    ▼
Script calls Python library (training_hub / sdg_hub / its_hub)
    │
    ▼
Result returned as JSON → presented to user
```

## Adding a New Plugin

1. Create the Python library with a public API
2. Create `scripts/_env.sh` and `<prefix>_detect.sh` first (foundation layer)
3. Create execution scripts that call the library's Python API
4. Create `commands/` markdown files referencing scripts via `${CLAUDE_PLUGIN_ROOT}`
5. Create `skills/` with `setup-guide` and one domain skill
6. Create all 4 agent manifests (copy from an existing plugin and adapt)
7. Add `.config-dir/` to `.gitignore`
9. Add entry to this marketplace's `.claude-plugin/marketplace.json`
10. Update the README status from "Coming soon" to "Ready"
