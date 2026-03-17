# Remove an Entry from the Library

## Context
The user wants to remove a skill, agent, or prompt from the library catalog and optionally delete the local copy.

## Input
The user provides a skill name or description.

## Steps

### 1. Sync the Library Repo
Pull the latest catalog before modifying:
```bash
cd <LIBRARY_SKILL_DIR>
git pull
```

### 2. Find the Entry
- Read `library.yaml`
- Search across all sections for the matching entry
- Determine the type (skill, agent, or prompt)
- If no match, tell the user the item wasn't found in the catalog

### 3. Confirm with User
Show the entry details and ask:
- "Remove **<name>** from the library catalog?"
- If installed locally, also ask: "Also delete the local copy at `<path>`?"

### 4. Remove from library.yaml
- Remove the entry from the appropriate section (`library.skills`, `library.agents`, or `library.prompts`)
- If other entries depend on this one (via `requires`), warn the user before proceeding

### 5. Delete Local Copy (if requested)

If the user confirmed local deletion:

1. Check all known directories for the entry:
   - Global: `~/.agents/skills/<name>`
   - Default: `.agents/skills/<name>` (relative to cwd)
   - Claude backward-compat: `~/.claude/skills/<name>`

2. Remove the canonical copy:
   ```bash
   rm -rf ~/.agents/skills/<name>
   ```

3. Remove platform symlinks. For each platform in `platforms` section of `library.yaml`:
   ```bash
   # Only remove if it's a symlink (not a real directory someone manually created)
   [ -L <platform_skills_dir>/<name> ] && rm <platform_skills_dir>/<name>
   ```

4. Remove from any other locations found in step 1.

### 6. Commit and Push
```bash
cd <LIBRARY_SKILL_DIR>
git add library.yaml
git commit -m "library: removed <type> <name>"
git push
```

### 7. Confirm
Tell the user:
- The entry has been removed from the catalog
- Whether the local copy was also deleted
- Which platform symlinks were removed
- If other entries depended on it, remind them to update those entries
