# Push a Skill to the Library Source

## Context
The user has improved a skill locally and wants to push changes back to the source.

## Input
The user provides a skill name or description.

## Steps

### 1. Resolve Target Catalog and Find the Entry

Determine which `library.yaml` to read:

1. Check if `./library.yaml` exists in the current working directory.
2. If it exists **and** the current directory is NOT `<LIBRARY_SKILL_DIR>`:
   - Use the local `./library.yaml`.
   - Set `<TARGET_YAML>` = `./library.yaml`
3. Otherwise, use the global catalog:
   - Set `<TARGET_YAML>` = `<LIBRARY_YAML_PATH>`

Then:
- Read `<TARGET_YAML>`
- Search across all sections for the matching entry
- If no match, tell the user the item wasn't found in the catalog

### 2. Locate the Local Copy
- Check the default directory for the type (from `default_dirs`)
- Check the global directory
- If found in multiple places, ask which one to push
- If not found locally, tell the user there's nothing to push

### 3. Check for Conflicts

**If source is a local path:**
- Compare the local installed copy with the source
- If the source has been modified since last pull, warn the user:
  "The source has changes that aren't in your local copy. Pushing will overwrite them. Continue?"

**If source is a GitHub URL:**
- Clone the repo to a temp directory (shallow):
  ```bash
  tmp_dir=$(mktemp -d)
  git clone --depth 1 --branch <branch> <clone_url> "$tmp_dir"
  ```
- Compare the skill directory in the clone with the local copy
- If they differ AND the remote has changes not in the local copy, warn about conflict
- Ask the user to resolve before continuing

### 4. Push to Source

**If source is a local path:**
- Copy the entire local directory to the source location, overwriting:
  ```bash
  cp -R <local_directory>/ <source_parent_directory>/
  ```
- Confirm the overwrite

**If source is a GitHub URL:**
- If we don't already have a tmp clone from step 3, clone now:
  ```bash
  tmp_dir=$(mktemp -d)
  git clone --depth 1 --branch <branch> <clone_url> "$tmp_dir"
  ```
- Remove the old skill directory in the clone:
  ```bash
  rm -rf "$tmp_dir/<skill_path_in_repo>"
  ```
- Copy the local version into the clone:
  ```bash
  cp -R <local_directory>/ "$tmp_dir/<skill_path_in_repo>/"
  ```
- Stage ONLY the relevant changes:
  ```bash
  cd "$tmp_dir"
  git add <skill_path_in_repo>
  ```
- Commit with the standard format:
  ```bash
  git commit -m "library: updated <name> <brief description of what changed>"
  ```
- Push:
  ```bash
  git push
  ```
- Clean up:
  ```bash
  rm -rf "$tmp_dir"
  ```

### 5. Confirm
Tell the user:
- What was pushed and where
- The commit message used
- If it was a local path push, confirm the overwrite
