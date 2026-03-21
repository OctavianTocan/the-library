# Use a Skill from the Library

## Context
Pull a skill, agent, or prompt from the catalog into the local environment. If already installed locally, overwrite with the latest from the source (refresh).

## Input
The user provides a skill name, repo source (`user/repo` or GitHub repo URL), or description, and optionally a target modifier.

## Steps

### 1. Resolve Target Catalog

Determine which `library.yaml` to read from:

1. Check if `./library.yaml` exists in the current working directory.
2. If it exists **and** the current directory is NOT `<LIBRARY_CATALOG_DIR>`:
   - Use the local `./library.yaml` as the catalog (per-repo manifest).
   - Set `<TARGET_YAML>` = `./library.yaml`
   - Set `<IS_LOCAL>` = true
3. Otherwise, use the global catalog:
   - Set `<TARGET_YAML>` = `<LIBRARY_YAML_PATH>`
   - Set `<IS_LOCAL>` = false

### 2. Find the Entry or Discover from Repo

**If the input is a repo-level source** (matches `user/repo` shorthand or `https://github.com/org/repo` without `/blob/`):
1. Run the repo discovery flow (see SKILL.md "Repo Discovery Flow" section)
2. Present all discovered SKILL.md files with their name + description from frontmatter
3. Let the user pick one or more
4. Resolve each to a full GitHub browser URL
5. For each selected skill, check if it already exists in `library.yaml` by name
   - If it exists, proceed to step 3 (dependencies) using the existing entry
   - If it doesn't exist, auto-register it in `library.yaml` (same as `/library add` steps 6-7: add entry, commit, push)
6. Then continue to step 3 for installation

**If the input is a name or description** (existing catalog lookup):
- Read `<TARGET_YAML>`
- Search across `library.skills`, `library.agents`, and `library.prompts`
- Match by name (exact) or description (fuzzy/keyword match)
- If multiple matches, show them and ask the user to pick one
- If no match, tell the user and suggest `/library search`

### 3. Resolve Dependencies
If the entry has a `requires` field:
- For each typed reference (`skill:name`, `agent:name`, `prompt:name`):
  - Look it up in `library.yaml`
  - If found, recursively run the `use` workflow for that dependency first
  - If not found, warn the user: "Dependency <ref> not found in library catalog"
- Process all dependencies before the requested item

### 4. Determine Target Directory

Read `default_dirs` and `platforms` from `library.yaml`. Parse the user's intent:

- **"globally" or "all platforms":** Use the `global` path (`~/.agents/skills/`). After copying, create platform symlinks (step 7).
- **"for cursor" / "for codex" / "for cline":** Use that platform's directory from `platforms` section. No symlinks.
- **Custom path specified:** Use that path. No symlinks.
- **No modifier (default):** Use the `default` path (`.agents/skills/` relative to cwd, project-local). No symlinks.

Select the correct section based on type (skills/agents/prompts).

### 5. Fetch from Source

**If source is a local path** (starts with `/` or `~`):
- Resolve `~` to the home directory
- Get the parent directory of the referenced file
- For skills: copy the entire parent directory to the target:
  ```bash
  cp -R <parent_directory>/ <target_directory>/<name>/
  ```
- For agents: copy just the agent file to the target:
  ```bash
  cp <agent_file> <target_directory>/<agent_name>.md
  ```
- For prompts: copy just the prompt file to the target:
  ```bash
  cp <prompt_file> <target_directory>/<prompt_name>.md
  ```
- If the agent or prompt is nested in a subdirectory under the `agents/` or `commands/` directories, copy the subdirectory to the target as well, creating the subdir if it doesn't exist. This keeps agents or commands grouped together.

**If source is a GitHub URL**:
- Parse the URL to extract: `org`, `repo`, `branch`, `file_path`
  - Browser URL pattern: `https://github.com/<org>/<repo>/blob/<branch>/<path>`
  - Raw URL pattern: `https://raw.githubusercontent.com/<org>/<repo>/<branch>/<path>`
- Determine the clone URL: `https://github.com/<org>/<repo>.git`
- Determine the parent directory path within the repo (everything before the filename)
- Clone into a temporary directory:
  ```bash
  tmp_dir=$(mktemp -d)
  git clone --depth 1 --branch <branch> https://github.com/<org>/<repo>.git "$tmp_dir"
  ```
- Copy the parent directory of the file to the target:
  ```bash
  cp -R "$tmp_dir/<parent_path>/" <target_directory>/<name>/
  ```
- Clean up:
  ```bash
  rm -rf "$tmp_dir"
  ```

**If clone fails (private repo)**, try SSH:
  ```bash
  git clone --depth 1 --branch <branch> git@github.com:<org>/<repo>.git "$tmp_dir"
  ```

### 6. Verify Installation
- Confirm the target directory exists
- Confirm the main file (SKILL.md, AGENT.md, or prompt file) exists in it
- Report success with the installed path

### 7. Create Platform Symlinks (global installs only)

Skip this step if the install was not global.

Read the `platforms` section from `library.yaml`. For each platform:

1. Resolve the platform's skills directory (e.g., `~/.claude/skills/`)
2. Check if the **parent** directory exists (e.g., `~/.claude/`). If not, skip this platform (not installed).
3. Create the skills subdirectory if needed: `mkdir -p <platform_skills_dir>`
4. Create a relative symlink:
   ```bash
   ln -sfn ../../.agents/skills/<name> <platform_skills_dir>/<name>
   ```

The relative path `../../.agents/skills/<name>` works because all platform skill dirs are at `~/.<platform>/skills/`, which is two levels up from the home directory to reach `~/.agents/skills/`.

Report which platforms got symlinks.

### 8. Confirm
Tell the user:
- What was installed and where
- Any dependencies that were also installed
- If this was a refresh (overwrite), mention that
- If global: which platform symlinks were created
