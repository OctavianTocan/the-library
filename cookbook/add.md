# Add a New Entry to the Library

## Context
Register a new skill, agent, or prompt in the library catalog.

## Input
The user provides: name, description, source, and optionally type and dependencies.

## Steps

### 1. Resolve Target Catalog

Determine which `library.yaml` to modify:

1. Check if `./library.yaml` exists in the current working directory.
2. If it exists **and** the current directory is NOT `<LIBRARY_CATALOG_DIR>`:
   - Use the local `./library.yaml` as the target catalog (per-repo manifest).
   - Set `<TARGET_YAML>` = `./library.yaml`
   - Set `<IS_LOCAL>` = true
3. Otherwise, use the global catalog:
   - Set `<TARGET_YAML>` = `<LIBRARY_YAML_PATH>`
   - Set `<IS_LOCAL>` = false

### 2. Detect Source Type and Discover

Determine what kind of source the user provided:

**Repo-level source** (no file path - matches `user/repo` or `https://github.com/org/repo` without `/blob/`):
1. Run the repo discovery flow (see SKILL.md "Repo Discovery Flow" section)
2. Present all discovered SKILL.md files with their name + description from frontmatter
3. Let the user pick one or more
4. Resolve each to a full GitHub browser URL
5. For each selected skill, continue to step 3 with the resolved URL
6. If multiple skills are selected, repeat steps 3-8 for each one

**File-level source** (local path, GitHub browser URL, or raw URL):
- Determine the type from the source path or user's prompt:
  - Source contains `SKILL.md` or user says "skill" -> type is `skill`
  - Source contains `AGENT.md` or user says "agent" -> type is `agent`
  - User says "prompt" -> type is `prompt`
  - If ambiguous, ask the user

### 3. Validate the Source
- **Local path**: Verify the file exists at the given path
- **GitHub file URL**: Verify the URL is well-formed (matches browser or raw URL patterns)
- Confirm the source points to a specific file, not a directory

### 4. Check Shareability

If the source is a local path (starts with `/` or `~`):

1. **Warn:** "Local paths don't work for teammates. Consider using a GitHub URL instead."

2. **Auto-detect GitHub URL:** Check if the local path is inside a git repo with a GitHub remote:
   ```bash
   cd <parent_directory_of_source>
   git remote get-url origin 2>/dev/null
   ```

3. If a GitHub remote is found:
   - Determine the relative path of the source file within the repo
   - Get the current branch: `git branch --show-current`
   - Construct the GitHub browser URL: `https://github.com/<org>/<repo>/blob/<branch>/<relative_path>`
   - Offer: "Found GitHub remote. Use this URL instead? `<constructed_url>`"
   - If user accepts, replace the source with the GitHub URL

4. If no remote found or user declines:
   - Proceed with the local path
   - Add a reminder: "You can update the source later with a GitHub URL for sharing."

### 5. Parse Dependencies
Detect dependencies by looking through the skill/agent/prompt files, format them as typed references:
- `skill:name`, `agent:name`, `prompt:name`
- Verify each dependency already exists in `library.yaml` or warn the user
  - If they don't exist add them to `library.yaml` first. If those files have dependencies, add them recursively.
  - You can detect these sometimes by looking at the frontmatter, and then in the file content look for `/<prompt|agent|skill>:name` references. If you're not sure, ask the user if they have any dependencies.

### 6. Add the Entry to the Target Catalog
Read `<TARGET_YAML>`, add the new entry under the correct section:

```yaml
# Under library.skills, library.agents, or library.prompts
- name: <name>
  description: <description>
  source: <source>
  requires: [<typed:refs>]  # omit if no dependencies
```

**YAML formatting rules:**
- 2-space indentation
- List items use `- ` prefix
- Properties are indented under the list item
- Keep entries alphabetically sorted by name within each section
- For skills reference the `.../<skill-name>/SKILL.md` file,
- For agents reference the `.../<agent name>.md` file,
- For prompts reference the `.../<prompt name>.md` file (installed to `.agents/commands/`),
- Remember we'll be adding an absolute path or a github url (https or ssh)

### 7. Confirm
Tell the user:
- The entry has been added and is now available via `/library use <name>`.
- If `<IS_LOCAL>`, mention it was added to the **project catalog** (`./library.yaml`), not the global one.
