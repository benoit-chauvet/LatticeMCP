# LatticeMCP

A [Claude Code](https://claude.com/claude-code) skill that keeps a team roadmap in Google Sheets and team priorities in [Lattice](https://lattice.com) in sync. It drives a real Firefox Nightly browser through the [Firefox DevTools MCP server](https://github.com/mozilla/firefox-devtools-mcp), so it needs no Lattice or Google API access.

## What it does

The `fill-roadmap` skill can sync in either direction:

- **Roadmap → Lattice:** reads the roadmap sheet, creates the priorities that are missing in Lattice, and fixes fields (dates, parent) on the ones that already exist.
- **Lattice → roadmap:** appends the Lattice priorities that are missing from the sheet.

Before changing anything, it shows the planned changes and waits for your approval. After writing, it reads the result back to check it.

## Requirements

- Claude Code
- Firefox Nightly, at `/Applications/Firefox Nightly.app` (edit `.mcp.json` if it lives elsewhere)
- Node.js / `npx`, which runs the Firefox DevTools MCP server
- A Lattice account

## Setup

1. Clone the repo and open it in Claude Code. The project's `.mcp.json` registers the `firefox-devtools` MCP server.
2. Create your URLs file from the template:
   ```sh
   cp .claude/skills/fill-roadmap/urls.template.md .claude/skills/fill-roadmap/urls.md
   ```
3. In `urls.md`, fill in:
   - **Team priorities (Lattice):** the Lattice Priorities "Explore" view, filtered to the owner and cycle you want.
   - **Roadmap file (Google Sheet):** the roadmap spreadsheet.

   `urls.md` is git-ignored, so your internal links stay local.

## Roadmap sheet format

| Type | Parent | Title | Start | End |
|---|---|---|---|---|
| Team prior | Autowebcompat | UA sniffing detection | 2026/05/01 | 2026/09/30 |
| Individual exp | Improve the use of QA | Merge reports into a single unified backlog | 2026/07/01 | 2026/12/31 |

- **Type:** `Team prior` = Lattice "Dept Goals/Team Prior"; `Individual exp` = "Individual Expectations".
- **Parent:** title of the parent priority in Lattice.
- **Start / End:** dates as `yyyy/mm/dd`.

The header sits on row 3, with data from row 4. The sheet must be readable through its link, and editable through it if you sync Lattice → roadmap.

## Usage

In Claude Code, run:

```
/fill-roadmap
```

You can also ask in plain words, for example "update the roadmap file with all my Lattice priorities".

On the first run, the MCP browser opens with its own profile. If Lattice asks you to sign in, sign in through SSO in that window; the skill never enters credentials. The browser stays open between runs so you stay logged in.

## Project layout

```
.mcp.json                                  # Firefox DevTools MCP server config
.claude/skills/fill-roadmap/SKILL.md       # Skill instructions, procedure and known gotchas
.claude/skills/fill-roadmap/urls.template.md
.claude/skills/fill-roadmap/urls.md        # Your URLs (git-ignored)
```
