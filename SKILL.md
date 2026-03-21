---
name: library
description: Private skill distribution system. Use when the user wants to install, use, add, push, remove, sync, list, search, or get help for skills, agents, or prompts from their private library catalog. Triggers on /library commands or mentions of library, skill distribution, or agentic management.
argument-hint: [command or prompt] [name or details]
---

# The Library

A meta-skill for private-first distribution of agentics (skills, agents, and prompts) across agents, devices, and teams.

## Variables

> Update these after forking and cloning the library repo.

- **LIBRARY_REPO_URL**: `https://github.com/OctavianTocan/the-library`
- **LIBRARY_YAML_PATH**: `~/.library/library.yaml`
- **LIBRARY_CATALOG_DIR**: `~/.library/`
- **LIBRARY_SKILL_DIR**: `~/.claude/skills/library/`

## How It Works

The Library is a catalog of references to your agentics. The `library.yaml` file points to where skills, agents, and prompts live (local filesystem or GitHub repos). Nothing is fetched until you ask for it.

**The `library.yaml` is a catalog, not a manifest.** Entries define what's *available* — not what gets installed. You pull specific items on demand with `/library use <name>`.

## Commands

| Command                     | Purpose                                  |
| --------------------------- | ---------------------------------------- |
| `/library help`             | Show available commands and usage         |
| `/library install`          | First-time setup, or install from per-repo yaml |
| `/library add <details>`    | Register a new entry in the catalog      |
| `/library use <name>`       | Pull from source (install or refresh)    |
| `/library push <name>`      | Push local changes back to source        |
| `/library remove <name>`    | Remove from catalog and optionally local |
| `/library list`             | Show full catalog with install status    |
| `/library sync`             | Re-pull all installed items from source   |
| `/library search <keyword>` | Find entries by keyword                  |

## Cookbook

Each command has a detailed step-by-step guide. **Read the relevant cookbook file before executing a command.**

| Command | Cookbook                                 | Use When                                                    |
| ------- | --------------------------------------- | ----------------------------------------------------------- |
| help    | [cookbook/help.md](cookbook/help.md)       | User asks for help, usage, or runs `/library` with no args  |
| install | [cookbook/install.md](cookbook/install.md) | First-time setup on a new device, or per-repo install       |
| add     | [cookbook/add.md](cookbook/add.md)         | User wants to register a new skill/agent/prompt in catalog  |
| use     | [cookbook/use.md](cookbook/use.md)         | User wants to pull or refresh a skill from the catalog      |
| push    | [cookbook/push.md](cookbook/push.md)       | User improved a skill locally and wants to update the source |
| remove  | [cookbook/remove.md](cookbook/remove.md)   | User wants to remove an entry from the catalog               |
| list    | [cookbook/list.md](cookbook/list.md)       | User wants to see what's available and what's installed      |
| sync    | [cookbook/sync.md](cookbook/sync.md)       | User wants to refresh all installed items at once            |
| search  | [cookbook/search.md](cookbook/search.md)   | User is looking for a skill but doesn't know the exact name |

**When a user invokes a `/library` command, read the matching cookbook file first, then execute the steps.**

## Source Format

The `source` field in `library.yaml` supports these formats (auto-detected):

- `/absolute/path/to/SKILL.md` — local filesystem
- `https://github.com/org/repo/blob/main/path/to/SKILL.md` — GitHub browser URL (file-level)
- `https://raw.githubusercontent.com/org/repo/main/path/to/SKILL.md` — GitHub raw URL (file-level)
- `user/repo` or `https://github.com/org/repo` — GitHub repo-level (auto-discovers skills)

Both GitHub URL formats are supported. Parse org, repo, branch, and file path from the URL structure. For private repos, use SSH or `GITHUB_TOKEN` for auth automatically.

**Important:** File-level sources point to a specific file (SKILL.md, AGENT.md, or prompt file). We always pull the entire parent directory, not just the file. Repo-level sources trigger auto-discovery (see below).

## Source Parsing Rules

**Local paths** start with `/` or `~`:
- Use the path directly. Copy the parent directory of the referenced file.

**GitHub file URLs** contain `/blob/` in the path:
- Browser URL pattern: `https://github.com/<org>/<repo>/blob/<branch>/<path>`
- Raw URL pattern: `https://raw.githubusercontent.com/<org>/<repo>/<branch>/<path>`
- Parse: `org`, `repo`, `branch`, `file_path`
- Clone URL: `https://github.com/<org>/<repo>.git`
- File location within repo: `<path>`

**GitHub repo-level sources** have no file path:
- Shorthand pattern: `<user>/<repo>` (no protocol, no `/blob/`)
- Full URL pattern: `https://github.com/<org>/<repo>` (no `/blob/` path)
- These trigger the **repo discovery flow** (see cookbook/add.md and cookbook/use.md)

## Repo Discovery Flow

When a repo-level source is provided (during `add` or `use`), discover available skills:

1. Detect default branch: `gh api repos/<user>/<repo> --jq .default_branch`
2. Find all SKILL.md files: `gh api repos/<user>/<repo>/git/trees/<branch>?recursive=1 --jq '.tree[].path' | grep 'SKILL.md$'`
3. For each SKILL.md, fetch frontmatter via `gh api repos/<user>/<repo>/contents/<path> --jq .content | base64 -d` to extract name + description
4. Present the list to the user, let them pick one or more
5. Resolve each selection to a full GitHub browser URL: `https://github.com/<user>/<repo>/blob/<branch>/<path>`
6. Store the resolved URL as the source in library.yaml (never store the repo-level shorthand)

## GitHub Workflow

When working with GitHub sources, prefer `gh api` for accessing single files (e.g., reading a SKILL.md to check metadata). For pulling entire skill directories, clone into a temp dir per the steps below.

**Fetching (use):**
1. Clone the repo with `git clone --depth 1 <clone_url>` into a temporary directory
2. Navigate to the parent directory of the referenced file
3. Copy that entire directory to the target local directory
4. The temporary directory is cleaned up automatically

**Pushing (push):**
1. Clone the repo with `git clone --depth 1 <clone_url>` into a temporary directory
2. Overwrite the skill directory in the clone with the local version
3. Stage only the relevant changes: `git add <skill_directory_path>`
4. Commit with message: `library: updated <skill name> <what changed>`
5. Push to remote
6. The temporary directory is cleaned up automatically

## Typed Dependencies

The `requires` field uses typed references to avoid ambiguity:
- `skill:name` — references a skill in the library catalog
- `agent:name` — references an agent in the library catalog
- `prompt:name` — references a prompt in the library catalog

When resolving dependencies: look up each reference in `library.yaml`, fetch all dependencies first (recursively), then fetch the requested item.

## Target Directories

By default, items are installed to the **default** directory from `library.yaml`:

```yaml
default_dirs:
    skills:
        - default: .agents/skills/
        - global: ~/.agents/skills/
        - claude: ~/.claude/skills/
    agents:
        - default: .agents/agents/
        - global: ~/.agents/agents/
    prompts:
        - default: .agents/commands/
        - global: ~/.agents/commands/
```

- If the user says "global" or "globally", use the `global` directory (`~/.agents/skills/`) and create platform symlinks.
- If the user says "for cursor" / "for codex" etc., install to that platform's directory only.
- If the user specifies a custom path, use that path.
- Otherwise, use the `default` directory (project-local).

## Cross-Platform Installation

The `.agents/skills/` directory is the open standard (Agent Skills, adopted by 16+ tools). When installing globally:

1. Copy skill to `~/.agents/skills/<name>/` (canonical store)
2. Detect installed platforms by checking which dirs exist from the `platforms` section in `library.yaml`
3. For each detected platform, create a relative symlink:
   ```bash
   ln -sfn ../../.agents/skills/<name> ~/.claude/skills/<name>
   ```
4. Report where installed and which platforms got symlinks

This matches the existing pattern where `~/.claude/skills/` and `~/.cursor/skills/` already symlink to `~/.agents/skills/`.

The `platforms` section in `library.yaml` is a lookup table:

```yaml
platforms:
  claude-code:
    skills: ~/.claude/skills/
  cursor:
    skills: ~/.cursor/skills/
  codex:
    skills: ~/.codex/skills/
  cline:
    skills: ~/.cline/skills/
```

Only platforms whose parent directory exists get symlinks. A missing `~/.cursor/` means Cursor isn't installed, so no symlink is created.

## Per-Repo Sharing

A `library.yaml` can live in any project repo, not just the library repo. This is the sharing mechanism.

**How it works:**
1. A project maintainer adds a `library.yaml` to their repo with GitHub URL sources
2. A teammate clones the repo and installs the library skill (one-time, via skills.sh or manually)
3. The teammate runs `/library install` in the project directory
4. The library skill detects the local `library.yaml` and installs all listed skills

**The library.yaml at `<LIBRARY_CATALOG_DIR>`** is the user's personal catalog (global).
**A library.yaml in a project repo** is a team manifest (project-scoped).

When the user runs `/library install`:
- If in a directory with a `library.yaml` that is NOT `<LIBRARY_CATALOG_DIR>`: install all entries from that yaml
- Otherwise: run first-time setup

**Important:** Per-repo library.yaml files should use GitHub URLs, not local paths. Local paths won't resolve on other machines.

## Catalog Location

The global catalog (`library.yaml`) lives at `<LIBRARY_CATALOG_DIR>` (`~/.library/`), separate from the skill code. This is a plain config directory, not a git repo. The skill code lives at `<LIBRARY_SKILL_DIR>`.

## Example Filled Library File

```yaml
default_dirs:
  skills:
    - default: .agents/skills/
    - global: ~/.agents/skills/
    - claude: ~/.claude/skills/
  agents:
    - default: .agents/agents/
    - global: ~/.agents/agents/
  prompts:
    - default: .agents/commands/
    - global: ~/.agents/commands/

platforms:
  claude-code:
    skills: ~/.claude/skills/
  cursor:
    skills: ~/.cursor/skills/
  codex:
    skills: ~/.codex/skills/
  cline:
    skills: ~/.cline/skills/

library:
  skills:
    - name: diagram-kroki
      description: Generate diagrams via Kroki HTTP API supporting 28+ languages
      source: https://github.com/myorg/private-skills/blob/main/skills/diagram-kroki/SKILL.md
      requires: [skill:firecrawl]

    - name: firecrawl
      description: Scrape, crawl, and search websites using Firecrawl CLI
      source: https://github.com/myorg/private-skills/blob/main/skills/firecrawl/SKILL.md

    - name: green-screen-captions
      description: Generate and burn AI-powered captions onto green screen videos
      source: https://github.com/myorg/video-tools/blob/main/skills/green-screen-captions/SKILL.md
      requires: [agent:video-processor, prompt:caption-style]

  agents:
    - name: code-reviewer
      description: Reviews code for quality, security, and performance
      source: https://github.com/myorg/agent-configs/blob/main/agents/code-reviewer/AGENT.md

    - name: video-processor
      description: Processes video files with ffmpeg and whisper transcription
      source: https://github.com/myorg/agent-configs/blob/main/agents/video-processor/AGENT.md

  prompts:
    - name: caption-style
      description: Style guide for generating video captions
      source: https://github.com/myorg/content/blob/main/prompts/caption-style.md

    - name: commit-message
      description: Standardized commit message format for all projects
      source: https://github.com/myorg/team-prompts/blob/main/prompts/commit-message.md
```
