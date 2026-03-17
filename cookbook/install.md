# Install The Library

## Context
First-time setup of The Library on a new device, or installing all skills from a per-repo `library.yaml`.

## Steps

### 1. Check Prerequisites
- Verify `git` is installed: `git --version`
- Verify the global skills directory exists or can be created: `~/.agents/skills/`

### 2. Detect Context

Check if there's a `library.yaml` in the current working directory.

**If a library.yaml exists and this is NOT the library repo** (check: `<LIBRARY_SKILL_DIR>` is different from current dir):
- This is a per-repo install. Switch to the per-repo workflow:
  1. Read the local `library.yaml`
  2. For each entry in `library.skills`, `library.agents`, and `library.prompts`, run the `use` workflow (see cookbook/use.md)
  3. Report what was installed
  4. Exit (skip the rest of install.md)

**If no library.yaml exists, or this IS the library repo:**
- Continue with first-time setup below.

### 3. Determine Fork Status
Ask the user: **"Is this the template repo or your own fork?"**

**If template repo (hasn't forked yet):**
- Instruct the user to create a private fork on GitHub
- Once forked, update the remote URL:
  ```bash
  cd <LIBRARY_SKILL_DIR>
  git remote set-url origin <fork_url>
  ```
- Verify with: `git remote -v`

**If already forked:**
- Skip this step

### 4. Clone to Global Skills Directory
If the repo isn't already cloned locally:
```bash
mkdir -p ~/.agents/skills/library
git clone <fork_url> ~/.agents/skills/library
```

If already cloned (e.g., user cloned the template first), just update the remote per step 3.

### 5. Update Variables
- Open `SKILL.md` in the library directory
- Take note of your current working directory.
- Update the `## Variables` section:
  - **LIBRARY_REPO_URL**: Set to the user's fork URL
  - **LIBRARY_YAML_PATH**: Confirm path (default: `~/.agents/skills/library/library.yaml`)
  - **LIBRARY_SKILL_DIR**: Confirm path (default: `~/.agents/skills/library/`)

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
- Confirm library.yaml exists alongside it
- Confirm symlinks exist for detected platforms
- Confirm the `/library` command is available

### 8. Done
Tell the user:
- The Library is now globally available via `~/.agents/skills/library/`
- Platform symlinks were created for: [list detected platforms]
- `/library list` will show the catalog
- `/library add` to start adding skills, agents, and prompts
- The `justfile` in the library directory has shorthand commands
