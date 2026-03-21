# Library Help

## Context
User asked for help, ran `/library` with no args, or ran `/library help`. Print the usage guide.

## Steps

### 1. Print the Help Output

Display this exactly:

```
The Library - private skill distribution for AI agents

Commands:
  /library help                Show this help message
  /library install             First-time setup, or install all skills from a per-repo library.yaml
  /library add <details>       Register a new skill, agent, or prompt in the catalog
  /library use <name>          Pull a skill from the catalog (install or refresh)
  /library push <name>         Push local changes back to the source
  /library remove <name>       Remove from catalog and optionally delete local copy
  /library list                Show full catalog with install status and platforms
  /library sync                Re-pull all installed items from their sources
  /library search <keyword>    Find entries by keyword

Modifiers for 'use':
  /library use <name> globally          Install to ~/.agents/skills/ + symlink to all platforms
  /library use <name> for cursor        Install to Cursor's skill directory only
  /library use <name> for codex         Install to Codex's skill directory only
  /library use <name> for cline         Install to Cline's skill directory only
  /library use <name> to <path>         Install to a custom path

Source formats (for 'add' and 'use'):
  /absolute/path/to/SKILL.md            Local filesystem
  https://github.com/org/repo/blob/...  GitHub file URL (direct to SKILL.md)
  https://raw.githubusercontent.com/... GitHub raw URL (direct to SKILL.md)
  user/repo                             GitHub repo shorthand (auto-discovers skills)
  https://github.com/org/repo           GitHub repo URL (auto-discovers skills)

Catalog: <LIBRARY_YAML_PATH>
Repo:    <LIBRARY_REPO_URL>
```

Replace `<LIBRARY_YAML_PATH>` and `<LIBRARY_REPO_URL>` with actual values from the Variables section in SKILL.md.

### 2. Done
That's it. No further action needed.
