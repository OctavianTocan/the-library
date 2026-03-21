# Library Skill: Cross-Platform Sharing Implementation Plan

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Make the library skill shareable across teams and platforms by formalizing the existing `~/.agents/skills/` symlink pattern and making per-repo `library.yaml` the sharing mechanism.

**Architecture:** The library.yaml becomes a portable manifest (like package.json for skills). Anyone with the library skill installed can clone a repo containing a library.yaml and install all listed skills. Global installs go to `~/.agents/skills/` (canonical) with relative symlinks to detected platform directories.

**Tech Stack:** YAML, Markdown, Bash (git, ln, cp)

---

## Context

The library skill works for personal use but breaks when sharing because:
1. Local paths in library.yaml don't transfer to other machines
2. No cross-platform support (Cursor, Codex, Cline all use different directories)

**Key discovery:** `~/.agents/skills/` is already the canonical store on this machine (96+ skills). `~/.claude/skills/` and `~/.cursor/skills/` already use relative symlinks like `../../.agents/skills/<name>`. The library just needs to formalize this existing pattern.

**Approach (revised):** Instead of bundling skills into the library repo, treat `library.yaml` as a portable manifest that can live in any repo. The library skill itself is installable via skills.sh. Teams share by committing a `library.yaml` with GitHub URLs to their project repo.

## What's NOT Changing

These files need no modifications:
- `cookbook/search.md` - already source-format agnostic
- `cookbook/push.md` - pushes to source, platform-agnostic
- `cookbook/sync.md` - re-pulls to target dir, symlinks stay valid since they point to canonical
- `justfile` - no new commands

## Files to Change

| File | Action | What changes |
|------|--------|--------------|
| `library.yaml` | MODIFY | Add `platforms` section, add `~/.agents/skills/` to `default_dirs` |
| `SKILL.md` | MODIFY | Cross-platform docs, per-repo sharing section, updated default_dirs, platforms docs |
| `cookbook/install.md` | MODIFY | Platform detection, symlink creation, per-repo yaml detection |
| `cookbook/add.md` | MODIFY | Shareability warning for local paths, GitHub URL auto-detect |
| `cookbook/use.md` | MODIFY | Platform-aware global install with symlinks, intent parsing |
| `cookbook/list.md` | MODIFY | Check `~/.agents/skills/`, show source type, show symlink status |
| `cookbook/remove.md` | MODIFY | Remove platform symlinks when removing global installs |

---

## Chunk 1: Schema and Documentation Foundation

### Task 1: Update library.yaml schema

**Files:**
- Modify: `~/.claude/skills/library/library.yaml`

- [ ] **Step 1: Update default_dirs to use agents paths**

Change `default_dirs` so the cross-platform standard is primary, with claude as backward compat:

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

- [ ] **Step 2: Add platforms section**

Add after `default_dirs`, before `library`:

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

This is a lookup table. Only platforms whose directories already exist on the machine get symlinks during install.

- [ ] **Step 3: Update the split-branch source to a GitHub URL**

The current source is a local path that won't work for anyone else. Change it to the GitHub URL (or leave it as-is and note that `/library add` will warn about this going forward). For now, leave it since the add warning will handle future entries.

- [ ] **Step 4: Verify yaml is valid**

```bash
python3 -c "import yaml; yaml.safe_load(open('library.yaml'))"
```

### Task 2: Update SKILL.md documentation

**Files:**
- Modify: `~/.claude/skills/library/SKILL.md`

- [ ] **Step 1: Update the default_dirs documentation in Target Directories section**

Replace the existing `default_dirs` code block (lines 113-124) with the new schema that includes `.agents/` paths and the `claude` backward-compat entry. Update the explanation bullets:

```markdown
## Target Directories

By default, items are installed to the **default** directory from `library.yaml`:

\`\`\`yaml
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
\`\`\`

- If the user says "global" or "globally", use the `global` directory (`~/.agents/skills/`) and create platform symlinks.
- If the user says "for cursor" / "for codex" etc., install to that platform's directory only.
- If the user specifies a custom path, use that path.
- Otherwise, use the `default` directory (project-local).
```

- [ ] **Step 2: Add Cross-Platform Installation section**

Add after Target Directories, before Library Repo Sync:

```markdown
## Cross-Platform Installation

The `.agents/skills/` directory is the open standard (Agent Skills, adopted by 16+ tools). When installing globally:

1. Copy skill to `~/.agents/skills/<name>/` (canonical store)
2. Detect installed platforms by checking which dirs exist from the `platforms` section in `library.yaml`
3. For each detected platform, create a relative symlink:
   ```bash
   ln -sfn ../../.agents/skills/<name> ~/.claude/skills/<name>
   ```
4. Report where installed and which platforms got symlinks

This matches the existing pattern on this machine where `~/.claude/skills/` and `~/.cursor/skills/` already symlink to `~/.agents/skills/`.

The `platforms` section in `library.yaml` is a lookup table:

\`\`\`yaml
platforms:
  claude-code:
    skills: ~/.claude/skills/
  cursor:
    skills: ~/.cursor/skills/
  codex:
    skills: ~/.codex/skills/
  cline:
    skills: ~/.cline/skills/
\`\`\`

Only platforms whose parent directory exists get symlinks. A missing `~/.cursor/` means Cursor isn't installed, so no symlink is created.
```

- [ ] **Step 3: Add Per-Repo Sharing section**

Add after Cross-Platform Installation, before Library Repo Sync:

```markdown
## Per-Repo Sharing

A `library.yaml` can live in any project repo, not just the library repo. This is the sharing mechanism.

**How it works:**
1. A project maintainer adds a `library.yaml` to their repo with GitHub URL sources
2. A teammate clones the repo and installs the library skill (one-time, via skills.sh or manually)
3. The teammate runs `/library install` in the project directory
4. The library skill detects the local `library.yaml` and installs all listed skills

**The library.yaml in the library repo** is the user's personal catalog (global).
**A library.yaml in a project repo** is a team manifest (project-scoped).

When the user runs `/library install`:
- If in a directory with a `library.yaml` that is NOT the library repo: install all entries from that yaml
- If in the library repo directory: run first-time setup

**Important:** Per-repo library.yaml files should use GitHub URLs, not local paths. Local paths won't resolve on other machines.
```

- [ ] **Step 4: Update the Example Filled Library File**

Update the example to use `.agents/` paths in default_dirs, include the `platforms` section, and make sure all example sources use GitHub URLs (not local paths) to model best practice for sharing:

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
    - name: video-processor
      description: Processes video files with ffmpeg and whisper transcription
      source: https://github.com/myorg/agent-configs/blob/main/agents/video-processor/AGENT.md

    - name: code-reviewer
      description: Reviews code for quality, security, and performance
      source: https://github.com/myorg/agent-configs/blob/main/agents/code-reviewer/AGENT.md

  prompts:
    - name: caption-style
      description: Style guide for generating video captions
      source: https://github.com/myorg/content/blob/main/prompts/caption-style.md

    - name: commit-message
      description: Standardized commit message format for all projects
      source: https://github.com/myorg/team-prompts/blob/main/prompts/commit-message.md
```

- [ ] **Step 5: Commit**

```bash
cd ~/.claude/skills/library
git add library.yaml SKILL.md
git commit -m "docs(library): add cross-platform and per-repo sharing docs"
```

---

## Chunk 2: Cookbook Updates - install and add

### Task 3: Update cookbook/install.md

**Files:**
- Modify: `~/.claude/skills/library/cookbook/install.md`

The install workflow gets two new capabilities: (1) platform detection with symlinks, and (2) per-repo yaml detection.

- [ ] **Step 1: Add per-repo detection to the top of the workflow**

Add a new step after "Check Prerequisites" and before "Determine Fork Status":

```markdown
### 2. Detect Context

Check if there's a `library.yaml` in the current working directory.

**If a library.yaml exists and this is NOT the library repo** (check: `<LIBRARY_SKILL_DIR>` is different from current dir):
- This is a per-repo install. Switch to the per-repo workflow:
  1. Read the local `library.yaml`
  2. For each entry, run the `use` workflow (see cookbook/use.md)
  3. Report what was installed
  4. Exit (skip the rest of install.md)

**If no library.yaml exists, or this IS the library repo:**
- Continue with first-time setup below.
```

- [ ] **Step 2: Add platform detection after cloning**

Add a new step after "Update Variables" (which becomes step 5):

```markdown
### 6. Detect Platforms and Create Symlinks

Check which agent platforms are installed:
```bash
for dir in ~/.claude ~/.cursor ~/.codex ~/.cline; do
  [ -d "$dir" ] && echo "$(basename $dir): installed"
done
```

For each detected platform that has a `skills/` subdirectory, create a symlink for the library skill:
```bash
# Example for Claude Code:
mkdir -p ~/.claude/skills
ln -sfn ../../.agents/skills/library ~/.claude/skills/library
```

Report which platforms were detected and linked.
```

- [ ] **Step 3: Update "Verify Installation" to check agents path**

Update the verification step to check `~/.agents/skills/library/` as the primary location:

```markdown
### 7. Verify Installation
- Confirm SKILL.md exists at `~/.agents/skills/library/SKILL.md` (or `<LIBRARY_SKILL_DIR>/SKILL.md`)
- Confirm library.yaml exists alongside it
- Confirm symlinks exist for detected platforms
- Confirm the `/library` command is available
```

- [ ] **Step 4: Renumber all steps and clean up**

Make sure the step numbers are sequential and consistent. The final order should be:
1. Check Prerequisites
2. Detect Context (new - per-repo detection)
3. Determine Fork Status
4. Clone to Global Skills Directory
5. Update Variables
6. Detect Platforms and Create Symlinks (new)
7. Verify Installation (updated)
8. Done

- [ ] **Step 5: Commit**

```bash
cd ~/.claude/skills/library
git add cookbook/install.md
git commit -m "feat(library): add platform detection and per-repo install to install workflow"
```

### Task 4: Update cookbook/add.md

**Files:**
- Modify: `~/.claude/skills/library/cookbook/add.md`

- [ ] **Step 1: Add shareability warning after "Validate the Source" (step 3)**

Insert a new step 4 between "Validate the Source" and "Parse Dependencies":

```markdown
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
   - Add a reminder: "Run `/library add` again with a GitHub URL when you want to share this."
```

- [ ] **Step 2: Renumber subsequent steps**

Steps after the new step 4 shift by one:
- Old step 4 (Parse Dependencies) becomes step 5
- Old step 5 (Add the Entry) becomes step 6
- Old step 6 (Commit and Push) becomes step 7
- Old step 7 (Confirm) becomes step 8

- [ ] **Step 3: Commit**

```bash
cd ~/.claude/skills/library
git add cookbook/add.md
git commit -m "feat(library): add shareability warning and GitHub URL auto-detect to add workflow"
```

---

## Chunk 3: Cookbook Updates - use (the big one)

### Task 5: Update cookbook/use.md

**Files:**
- Modify: `~/.claude/skills/library/cookbook/use.md`

This is the largest change. The `use` command gets platform-aware installation with symlinks and user intent parsing.

- [ ] **Step 1: Add user intent parsing to "Determine Target Directory" (step 4)**

Replace the current step 4 with expanded intent parsing:

```markdown
### 4. Determine Target Directory

Read `default_dirs` and `platforms` from `library.yaml`. Parse the user's intent:

- **"globally" or "all platforms":** Use the `global` path (`~/.agents/skills/`). After copying, create platform symlinks (step 6).
- **"for cursor" / "for codex" / "for cline":** Use that platform's directory from `platforms` section. No symlinks.
- **Custom path specified:** Use that path. No symlinks.
- **No modifier (default):** Use the `default` path (`.agents/skills/` - project-local). No symlinks.

Select the correct section based on type (skills/agents/prompts).
```

- [ ] **Step 2: Update "Fetch from Source" to install to agents path**

No structural change needed here. The fetch step already copies to whatever target directory was determined in step 4. The target just changed from `~/.claude/skills/` to `~/.agents/skills/` for global installs.

- [ ] **Step 3: Add new step 6 for platform symlinks**

Insert after "Verify Installation" (step 5 becomes 6, renumber):

```markdown
### 6. Create Platform Symlinks (global installs only)

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
```

- [ ] **Step 4: Update "Confirm" step to report symlinks**

Update the final confirmation to include:
```markdown
### 7. Confirm
Tell the user:
- What was installed and where
- Any dependencies that were also installed
- If this was a refresh (overwrite), mention that
- If global: which platform symlinks were created
```

- [ ] **Step 5: Renumber all steps**

Final order:
1. Sync the Library Repo
2. Find the Entry
3. Resolve Dependencies
4. Determine Target Directory (updated with intent parsing)
5. Fetch from Source (unchanged logic, new default target)
6. Create Platform Symlinks (new, global only)
7. Verify Installation
8. Confirm (updated)

- [ ] **Step 6: Commit**

```bash
cd ~/.claude/skills/library
git add cookbook/use.md
git commit -m "feat(library): add platform-aware global install with symlinks to use workflow"
```

---

## Chunk 4: Cookbook Updates - list and remove

### Task 6: Update cookbook/list.md

**Files:**
- Modify: `~/.claude/skills/library/cookbook/list.md`

- [ ] **Step 1: Update "Check Install Status" to check agents path and platform symlinks**

Replace step 3 with:

```markdown
### 3. Check Install Status

For each entry, determine install status:
- Check the `global` directory (`~/.agents/skills/<name>`)
- Check the `default` directory (`.agents/skills/<name>` relative to cwd)
- Check the `claude` backward-compat directory (`~/.claude/skills/<name>`)
- Mark as: `installed (global)`, `installed (default)`, `installed (claude)`, or `not installed`

For globally installed entries, also check which platform symlinks exist:
- For each platform in `platforms` section, check if `<platform_skills_dir>/<name>` is a symlink
- Collect the list of linked platforms
```

- [ ] **Step 2: Update "Display Results" to show source type and platforms**

Add a "Source Type" column and a "Platforms" column:

```markdown
### 4. Display Results

Determine source type for each entry:
- Source starts with `/` or `~` -> `local`
- Source starts with `https://github.com` -> `github`

Format the output as a table grouped by type:

\`\`\`
## Skills
| Name | Description | Source Type | Status | Platforms |
|------|-------------|------------|--------|-----------|
| skill-name | description | github | installed (global) | claude, cursor, cline |
| other-skill | description | local | not installed | - |

## Agents
...

## Prompts
...
\`\`\`

If a section is empty, show: `No <type> in catalog.`
```

- [ ] **Step 3: Commit**

```bash
cd ~/.claude/skills/library
git add cookbook/list.md
git commit -m "feat(library): show agents path, source type, and platform symlinks in list"
```

### Task 7: Update cookbook/remove.md

**Files:**
- Modify: `~/.claude/skills/library/cookbook/remove.md`

- [ ] **Step 1: Update "Delete Local Copy" to also remove platform symlinks**

Replace step 5 with:

```markdown
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
```

- [ ] **Step 2: Commit**

```bash
cd ~/.claude/skills/library
git add cookbook/remove.md
git commit -m "feat(library): remove platform symlinks when deleting global installs"
```

---

## Verification

After all tasks are complete:

1. **Schema validity:** Run `python3 -c "import yaml; yaml.safe_load(open('library.yaml'))"` to verify yaml is valid.

2. **No dead references:** Grep all files for `library://`, `artifacts/`, `publish` to confirm none of the dropped concepts leaked in.
   ```bash
   cd ~/.claude/skills/library
   grep -r "library://" . --include="*.md" --include="*.yaml"
   grep -r "artifacts/" . --include="*.md" --include="*.yaml"
   grep -r "publish" . --include="*.md" --include="*.yaml"
   ```

3. **Consistent terminology:** Verify all files reference `~/.agents/skills/` as the global path, not `~/.claude/skills/`.

4. **Step numbering:** Read through each modified cookbook file and verify step numbers are sequential with no gaps.

5. **SKILL.md coherence:** Read SKILL.md end-to-end and verify:
   - No references to removed concepts
   - Example library file matches the real schema
   - All cookbook links still work
   - default_dirs in docs matches default_dirs in library.yaml

6. **Manual smoke test:** Run `/library list` and verify the output reflects the new format.
