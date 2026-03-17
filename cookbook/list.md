# List Available Skills

## Context
Show the full library catalog with install status, source type, and platform symlinks.

## Steps

### 1. Sync the Library Repo
Pull the latest catalog before reading:
```bash
cd <LIBRARY_SKILL_DIR>
git pull
```

### 2. Read the Catalog
- Read `library.yaml`
- Parse all entries from `library.skills`, `library.agents`, and `library.prompts`

### 3. Check Install Status
For each entry:
- Check the `global` directory (`~/.agents/skills/<name>`)
- Check the `default` directory (`.agents/skills/<name>` relative to cwd)
- Check the `claude` backward-compat directory (`~/.claude/skills/<name>`)
- Mark as: `installed (global)`, `installed (default)`, `installed (claude)`, or `not installed`

For globally installed entries, also check which platform symlinks exist:
- For each platform in `platforms` section, check if `<platform_skills_dir>/<name>` is a symlink
- Collect the list of linked platforms

### 4. Display Results

Determine source type for each entry:
- Source starts with `/` or `~` -> `local`
- Source starts with `https://github.com` -> `github`

Format the output as a table grouped by type:

```
## Skills
| Name | Description | Source Type | Status | Platforms |
|------|-------------|------------|--------|-----------|
| skill-name | description | github | installed (global) | claude, cursor, cline |
| other-skill | description | local | not installed | - |

## Agents
| Name | Description | Source Type | Status | Platforms |
|------|-------------|------------|--------|-----------|
| agent-name | description | github | installed (global) | claude |

## Prompts
| Name | Description | Source Type | Status | Platforms |
|------|-------------|------------|--------|-----------|
| prompt-name | description | github | not installed | - |
```

If a section is empty, show: `No <type> in catalog.`

### 5. Summary
At the bottom, show:
- Total entries in catalog
- Total installed locally
- Total not installed
