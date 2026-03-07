# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

This is a **source** repository for [mycli](https://github.com/fernandonogueira/mycli) — a git-backed collection of command libraries. Users add it with `my source add <url>`, and commands become available as `my <library> <slug>`.

## Repository Structure

```
mycli.yaml              # Required manifest at repo root
<library>/              # One directory per library (e.g., "ops", "k8s")
  <slug>.yaml           # Command spec files (filename must match metadata.slug)
  <slug>.json           # JSON specs also accepted
```

Manifest filename priority: `mycli.yaml` > `mycli.yml` > `mycli.json` > `shelf.yaml` > `shelf.yml` > `shelf.json`

## Manifest Format (mycli.yaml)

```yaml
schemaVersion: 1
name: source-name
description: Optional description
namespace: optional-namespace
libraries:
  lib-key:
    name: Display Name
    description: Optional
    path: lib-key
    aliases:
      - short-name
```

- `schemaVersion` must be `1`
- `libraries` must have at least one entry
- Library keys must match `^[a-z][a-z0-9-]*$`
- `namespace` is optional
- Library `aliases` must match `^[a-z][a-z0-9-]*$` and be unique

## Command Spec Format

Each spec file (`.json`, `.yaml`, `.yml`) in a library directory must validate against the [command-v1 schema](https://github.com/fernandonogueira/mycli/blob/main/pkg/spec/schema/command-v1.schema.json). The filename (minus extension) **must match** the spec's `metadata.slug`.

Required fields: `schemaVersion: 1`, `kind: "command"`, `metadata` (with `name` and `slug`), `steps` (at least one).

Steps use Go template syntax: `{{.args.X}}`, `{{.env.X}}`, `{{.cwd}}`, `{{.home}}`.

### Metadata

- `metadata.tags` — string array for categorization
- `metadata.aliases` — alternative slugs (must match `^[a-z][a-z0-9-]*$`, unique)

### Dependencies

`dependencies` — array of executable names required to run the command (e.g., `["kubectl", "jq"]`). Must be unique.

### Defaults

Top-level `defaults` section sets defaults for all steps:

- `shell` — one of `/bin/sh`, `/bin/bash`, `/bin/zsh`, `/usr/bin/env`
- `timeout` — duration string (e.g., `"30s"`, `"5m"`, pattern: `^[0-9]+(ns|us|us|ms|s|m|h)+$`)
- `env` — key-value environment variables (keys must match `^[A-Za-z_][A-Za-z0-9_]*$`)
- `templateDelimiters` — custom left/right delimiters, array of exactly 2 strings (default: `["{{", "}}"]`)

### Steps

Each step requires `name` and `run`. Optional step-level fields:

- `env` — step-specific environment variables (same key pattern as defaults)
- `timeout` — step-specific timeout (same pattern as defaults)
- `continueOnError` — boolean, continue execution if step fails
- `shell` — step-specific shell override (same enum as defaults)

### Policy

Top-level `policy` section for execution constraints:

- `requireConfirmation` — boolean, prompt user before execution
- `allowedExecutables` — whitelist of executables the command may invoke

## Validation Rules

- Invalid specs are skipped with warnings (don't block the source from being added)
- Filename/slug mismatch causes the spec to be skipped
- Slugs must match `^[a-z][a-z0-9-]*$`
- Only top-level files in each library directory are discovered (no subdirectory recursion)

## Conventions

- Prefer YAML (`.yaml`) for command specs.
- Use `set -euo pipefail` at the top of multi-line bash steps.
- Set `policy.requireConfirmation: true` for destructive commands (delete, drop, restart, etc.).
- Use `/bin/bash` as the default shell when the script uses bash-specific features.
- Filename must match `metadata.slug` (e.g., `delete-pods.yaml` has slug `delete-pods`).

## Testing Locally

```bash
# Validate a spec using the mycli CLI
my cli run -f <library>/<slug>.yaml [args...]

# Add this source locally (from a local path or after pushing)
my source add <repo-url>

# Run a command
my <library> <slug> [args...]
```
