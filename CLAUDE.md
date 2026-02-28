# CLAUDE.md — AI Assistant Guide for obsidian-community-releases

## Repository Overview

This is the **official Obsidian community registry** repository. It is **not** the Obsidian source code (Obsidian is closed-source). Its purposes are:

- Host public releases of the Obsidian desktop application
- Maintain the community plugins directory (600+ plugins)
- Maintain the community CSS themes directory
- Track plugin download statistics
- Serve as the submission gateway for community contributions via pull requests

---

## Repository Structure

```
/
├── community-plugins.json            # Primary plugin registry (~19K lines)
├── community-plugins-removed.json   # Removed/deprecated plugins
├── community-plugin-deprecation.json # Per-plugin version deprecations
├── community-plugin-stats.json      # Download statistics (~57K lines)
├── community-css-themes.json        # Primary themes registry (~2.9K lines)
├── community-css-themes-removed.json # Removed themes
├── community-snippets.json          # Community code snippets
├── desktop-releases.json            # Obsidian desktop release metadata
├── dark.png / light.png             # Theme showcase screenshots
├── package.json                     # npm scripts (formatting only)
├── cla.md                           # Contributor License Agreement
├── plugin-review.md                 # Points to external plugin guidelines
├── README.md                        # Submission instructions
└── .github/
    ├── workflows/                   # GitHub Actions automation
    └── PULL_REQUEST_TEMPLATE/       # PR templates for plugins and themes
```

**There is no `src/` directory.** This repository is data-only — JSON files are the primary deliverable.

---

## Data File Formats

### `community-plugins.json`
An array of plugin objects. Each entry must have:
```json
{
  "id": "unique-plugin-id",
  "name": "Display Name",
  "author": "Author Name",
  "description": "Short description of what the plugin does",
  "repo": "github-user/repo-name"
}
```
- `id` must match the `id` in the plugin's `manifest.json`
- New entries are appended to the **end** of the array
- The `name`, `author`, and `description` fields are used by Obsidian's search

### `community-css-themes.json`
An array of theme objects. Each entry must have:
```json
{
  "name": "Theme Name",
  "author": "Author Name",
  "repo": "github-user/repo-name",
  "screenshot": "path/to/screenshot.png",
  "modes": ["dark", "light"]
}
```
- Optional fields: `"publish": true` (if theme supports Obsidian Publish), `"legacy": true`
- Screenshot must be 16:9 aspect ratio (validated by CI)
- `modes` must be `["dark"]`, `["light"]`, or `["dark", "light"]`
- New entries are appended to the **end** of the array

### `community-plugin-stats.json`
Maps plugin IDs to download counts per version:
```json
{
  "plugin-id": {
    "1.0.0": 5000,
    "downloads": 50000,
    "updated": 1700000000
  }
}
```
This file is updated automatically by the `plugin-stat.yml` GitHub Actions workflow daily. **Do not edit manually.**

### `desktop-releases.json`
Contains Obsidian desktop release metadata including version numbers, download URLs, and verification hashes. Updated by maintainers during release cycles.

---

## Development Commands

```bash
# Install dependencies
npm ci

# Format all community JSON files with Prettier
npm run format
```

There is **no build step**, **no test suite**, and **no compilation**. The only npm script is formatting.

---

## GitHub Actions Workflows

| Workflow | Trigger | Purpose |
|---|---|---|
| `format.yml` | Push to `master` | Auto-formats JSON with Prettier, auto-commits |
| `plugin-stat.yml` | Daily cron (00:00 UTC) | Fetches GitHub release stats, updates `community-plugin-stats.json` |
| `validate-plugin-entry.yml` | PR touching `community-plugins.json` | Validates JSON, required fields, repo existence |
| `validate-theme-entry.yml` | PR touching `community-css-themes.json` | Validates JSON, required fields, screenshot dimensions |
| `skip.yml` | `/skip` comment on PR | Adds "Skipped code scan" label for maintainer bypass |
| `stale.yml` | Daily cron (07:27 UTC) | Closes PRs inactive >30 days (warns at 15 days) |
| `stale-validation-failed.yml` | Stale detection | Handles PRs that failed validation |

---

## Commit Message Conventions

Follow the patterns used in this repository:

```
chore: Update plugin stats          # Automated stats commit
chore: Update community plugins     # Automated plugin list update
chore: Update community themes      # Automated theme list update
chore: Format JSON                  # Automated Prettier formatting
Update beta to vX.Y.Z               # Release version bump
Update public to vX.Y.Z             # Stable release bump
Add plugin: Plugin Name (#NNNN)     # Manual plugin acceptance
```

---

## Pull Request Workflow

All community submissions come through PRs. The process:

1. Contributor opens a PR using the `plugin.md` or `theme.md` PR template
2. Contributor must sign the CLA (`cla.md`) — respond "I have read the CLA agreement and I hereby sign the CLA"
3. GitHub Actions auto-validates the submission:
   - JSON structure and required fields
   - Repository accessibility on GitHub
   - Screenshot dimensions for themes (16:9)
   - Manifest.json compatibility for plugins
4. Maintainers review and merge approved submissions
5. `format.yml` auto-runs on master after merge

**Stale PR policy:** PRs inactive for 15 days receive a warning; PRs inactive for 30 days are auto-closed. Exempt labels: "Ready for review", "Skipped code scan".

---

## Key Policies and Links

- **Developer policies:** https://docs.obsidian.md/Developer+policies
- **Plugin submission guide:** https://docs.obsidian.md/Plugins/Releasing/Submit+your+plugin
- **Plugin guidelines:** https://docs.obsidian.md/Plugins/Releasing/Plugin+guidelines
- **Theme submission guide:** https://docs.obsidian.md/Themes/App+themes/Submit+your+theme
- **Theme guidelines:** https://docs.obsidian.md/Themes/App+themes/Theme+guidelines
- **Community forum:** https://forum.obsidian.md
- **Community Discord:** https://obsidian.md/community

---

## AI Assistant Guidelines

### What this repo is for
- Reviewing and modifying plugin/theme registry entries
- Updating release metadata in `desktop-releases.json`
- Editing CI/CD workflow files in `.github/workflows/`
- Updating documentation files

### What to avoid
- **Never** edit `community-plugin-stats.json` by hand — it is auto-maintained by GitHub Actions
- **Do not** add entries to the middle of `community-plugins.json` or `community-css-themes.json` — always append to the end
- **Do not** introduce JavaScript/TypeScript source files; this is a data-only repository
- **Do not** remove or reorder existing plugin/theme entries without a clear reason; ordering affects presentation

### JSON editing rules
- Run `npm run format` after any edits to community JSON files to ensure Prettier formatting is applied
- Validate that IDs are unique before adding new plugin entries
- When removing a plugin/theme, move its entry to the corresponding `*-removed.json` file with a removal reason

### Formatting
- All JSON files use Prettier 3.5.3 with default settings
- 2-space indentation, trailing commas where supported

### Git
- All commits in this repository are GPG/SSH signed
- Branch naming for automated tasks: `claude/<task-id>`
- Target branch for merges: `master`
