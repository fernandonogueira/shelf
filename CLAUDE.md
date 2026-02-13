# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

This is a **shelf** repository for [mycli](https://github.com/fernandonogueira/mycli) — a git-backed collection of command libraries. Users add it with `my shelf add <url>`, and commands become available as `my <library> <slug>`.

## Repository Structure

```
shelf.json              # Required manifest at repo root
<library>/              # One directory per library (e.g., "ops", "k8s")
  <slug>.json           # Command spec files (filename must match metadata.slug)
  <slug>.yaml           # YAML specs also accepted (.yaml, .yml)
```

## Manifest Format (shelf.json)

```json
{
  "shelfVersion": 1,
  "name": "shelf-name",
  "description": "Optional description",
  "libraries": {
    "lib-key": {
      "name": "Display Name",
      "description": "Optional",
      "path": "lib-key"
    }
  }
}
```

- `shelfVersion` must be `1`
- `libraries` must have at least one entry
- Library keys must match `^[a-z][a-z0-9-]*$`

## Command Spec Format

Each spec file (`.json`, `.yaml`, `.yml`) in a library directory must validate against the [command-v1 schema](https://github.com/fernandonogueira/mycli/blob/main/pkg/spec/schema/command-v1.schema.json). The filename (minus extension) **must match** the spec's `metadata.slug`.

Required fields: `schemaVersion: 1`, `kind: "command"`, `metadata` (with `name` and `slug`), `steps` (at least one).

Steps use Go template syntax: `{{.args.X}}`, `{{.env.X}}`, `{{.cwd}}`, `{{.home}}`.

## Validation Rules

- Invalid specs are skipped with warnings (don't block the shelf from being added)
- Filename/slug mismatch causes the spec to be skipped
- Slugs must match `^[a-z][a-z0-9-]*$`
- Only top-level files in each library directory are discovered (no subdirectory recursion)

## Testing Locally

```bash
# Validate a spec using the mycli CLI
my cli run -f <library>/<slug>.json [args...]

# Add this shelf locally (from a local path or after pushing)
my shelf add <repo-url>

# Run a shelf command
my <library> <slug> [args...]
```
