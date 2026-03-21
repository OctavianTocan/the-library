# Install The Library

## Context
First-time setup of The Library on a new device, or installing all skills from a per-repo `library.yaml`.

## Steps

### 1. Check Prerequisites
- Verify `git` is installed: `git --version`
- Verify the global skills directory exists or can be created: `~/.agents/skills/`

### 2. Detect Context

Check if there's a `library.yaml` in the current working directory.

**If a library.yaml exists and this is NOT `<LIBRARY_CATALOG_DIR>`:**
- This is a per-repo install. Switch to the per-repo workflow:
  1. Read the local `library.yaml`
  2. For each entry in `library.skills`, `library.agents`, and `library.prompts`, run the `use` workflow (see cookbook/use.md)
  3. Report what was installed
  4. Exit (skip the rest of install.md)

**Otherwise:**
- Continue with first-time setup below.

### 3. Initialize the Global Catalog

Create the catalog directory and initialize `library.yaml` if it doesn't exist:
```bash
mkdir -p ~/.library
```

**If `~/.library/library.yaml` already exists:**
- Skip initialization, the catalog is already set up.

**If it doesn't exist but `<LIBRARY_SKILL_DIR>/library.yaml` exists (migration from old layout):**
- Copy the existing catalog: `cp <LIBRARY_SKILL_DIR>/library.yaml ~/.library/library.yaml`
- Tell the user: "Migrated your existing catalog to `~/.library/library.yaml`."

**If neither exists:**
- Create a fresh catalog with the default structure (see SKILL.md "Example Filled Library File" for the template, but with empty `library.skills`, `library.agents`, and `library.prompts` lists).

### 4. Update Variables
- Open `SKILL.md` in the library skill directory
- Update the `## Variables` section:
  - **LIBRARY_REPO_URL**: Set to the user's fork URL (if they have one)
  - **LIBRARY_YAML_PATH**: Confirm path (default: `~/.library/library.yaml`)
  - **LIBRARY_CATALOG_DIR**: Confirm path (default: `~/.library/`)
  - **LIBRARY_SKILL_DIR**: Confirm path (default: the current skill directory)

### 6. Detect Platforms and Create Symlinks

Check which agent platforms are installed:
```bash
for dir in ~/.claude ~/.cursor ~/.codex ~/.cline; do
  [ -d "$dir" ] && echo "$(basename $dir): installed"
done
```

For each detected platform that has a `skills/` subdirectory (or create it), symlink the library skill:
```bash
# Example for Claude Code:
mkdir -p ~/.claude/skills
ln -sfn ../../.agents/skills/library ~/.claude/skills/library
```

Repeat for each detected platform. Report which platforms were detected and linked.

### 7. Verify Installation
- Confirm SKILL.md exists at `<LIBRARY_SKILL_DIR>/SKILL.md`
- Confirm library.yaml exists at `<LIBRARY_YAML_PATH>`
- Confirm symlinks exist for detected platforms
- Confirm the `/library` command is available

### 8. Done
Tell the user:
- The Library skill is installed at `<LIBRARY_SKILL_DIR>`
- The global catalog is at `~/.library/library.yaml`
- Platform symlinks were created for: [list detected platforms]
- `/library list` will show the catalog
- `/library add` to start adding skills, agents, and prompts
