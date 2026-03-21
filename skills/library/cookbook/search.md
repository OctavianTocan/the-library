# Search the Library

## Context
Find entries in the catalog by keyword when the user doesn't remember the exact name.

## Input
The user provides a keyword or description.

## Steps

### 1. Resolve Target Catalog

Determine which `library.yaml` to search:

1. Check if `./library.yaml` exists in the current working directory.
2. If it exists **and** the current directory is NOT `<LIBRARY_CATALOG_DIR>`:
   - Use the local `./library.yaml`.
   - Set `<TARGET_YAML>` = `./library.yaml`
3. Otherwise, use the global catalog:
   - Set `<TARGET_YAML>` = `<LIBRARY_YAML_PATH>`

### 2. Read the Catalog
- Read `<TARGET_YAML>`
- Parse all entries from `library.skills`, `library.agents`, and `library.prompts`

### 3. Search
- Match the keyword (case-insensitive) against:
  - Entry `name`
  - Entry `description`
- A match is any entry where the keyword appears as a substring in either field
- Collect all matches across all types

### 4. Display Results

If matches found, format as:

```
## Search Results for "<keyword>"

| Type | Name | Description | Source |
|------|------|-------------|--------|
| skill | matching-skill | description... | source... |
| agent | matching-agent | description... | source... |
```

If no matches:
```
No results found for "<keyword>".

Tip: Try broader keywords or run `/library list` to see the full catalog.
```

### 5. Suggest Next Step
If matches were found, suggest: `Run /library use <name> to install one of these.`
