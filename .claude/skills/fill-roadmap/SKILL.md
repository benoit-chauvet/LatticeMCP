---
name: fill-roadmap
description: Sync team priorities from the roadmap Google Sheet into Lattice (create missing priorities, fix fields on existing ones) using the Firefox devtools MCP. Use when the user asks to fill, sync or update the Lattice roadmap/priorities.
---

# Fill Roadmap

## Description

- Use devtools MCP with Firefox Nightly
- Add team priorities described in the roadmap file
- If a priority already exists, check the fields and update if necessary

## Browser session

The firefox-devtools server uses a single persistent profile (`--autoProfile`, keeps Google/Lattice logins). Firefox allows only one instance per profile, so if another Claude session already has the MCP Firefox open, launching here fails with `Process (pid=…) unexpectedly closed with status 0`.

- **Always reuse the same browser instance.** Start with `list_pages` and work in the already-open tab with `navigate_page`. Do not open new tabs (`new_page`), do not relaunch Firefox, and do **not** call `close_firefox_session` at the end — leave the instance and tab open for the next run.
- If the launch error above occurs, stop and ask the user to close the other session's MCP Firefox window, then retry.
- If Lattice shows "Sign back in to Mozilla", ask the user to log in via SSO in that window — never enter credentials.

## Parameters

- devtools config : .mcp.json
- Team priorities and roadmap file URLs: read [urls.md](urls.md) at the start of each run. It is git-ignored; if it's missing, copy [urls.template.md](urls.template.md) to `urls.md` and ask the user for the URLs.

## Procedure

1. **Read the roadmap.** Navigate to the roadmap file, then fetch it as CSV from the page (same-origin, works without Google login since the sheet is public):
   `fetch('/spreadsheets/d/<SHEET_ID>/gviz/tq?tqx=out:csv&gid=<GID>')`, with the sheet ID and gid taken from the roadmap URL in urls.md.
   (`/export?format=csv` fails with a NetworkError — cross-origin redirect.) Columns: Type, Parent, Title, Start, End. "Team prior" = Lattice type "Dept Goals/Team Prior".
2. **Read Lattice.** Navigate to the team priorities URL. Rows are `[role=group]` elements; click every `button[aria-label="Expand row"]` repeatedly until none remain, then list titles. Clicking a title opens its details in the `aside` (type, start, due, owners, cycle, parent).
3. **Diff and confirm.** Show the user which priorities will be created and which fields will change. Get approval before writing anything.
4. **Update existing:** in the `aside`, open the `button[aria-haspopup=menu]` → "Edit" → change fields → "Save".
5. **Create missing:** select the parent priority, then click "Add Dept Goals/Team Prior" in its `aside` — this pre-fills parent, cycle, owner, Individual, Private. Set title, dates, then click the "Create" button next to "Save draft" (not the top-nav "Create").
6. **Verify** by reopening each touched priority and checking its details.

## Writing to the roadmap sheet (Lattice → sheet)

- The sheet is editable without a Google login — no need to ask the user to sign in.
- Layout: header on **row 3**, data from row 4 (the CSV export hides the two blank top rows, so don't infer row numbers from it). Find the first empty row with a screenshot or the formula bar before writing.
- Type labels: "Team prior" (Dept Goals/Team Prior), "Individual exp" (Individual Expectations). Dates as `yyyy/mm/dd`.
- To write: turn off Outils → Commandes des suggestions → "Activer la saisie semi-automatique" (otherwise Tab/Enter can accept a longer autocompleted title), select the start cell via the Name box (`#t-name-box`: click, type `A13`, Enter), then `type_text` **one cell per call** with `submitKey: "Tab"`, and `"Enter"` on the last column (returns to column A of the next row). Re-enable autocomplete afterwards.
- Don't paste: synthetic paste events are ignored, and Cmd+V of TSV lands as one multi-line value in a single cell. Don't put `\t`/`\n` inside `type_text` either — they're typed literally into the cell. Escape discards an uncommitted cell edit; if something got committed, select the cell and press Delete.
- Verify by re-fetching the gviz CSV (add a cache-busting query param).

## Gotchas

- **Dates:** typing into the date inputs doesn't stick. Click the input to open the react-day-picker (`.DayPicker`), navigate with `.DayPicker-NavButton--prev/--next` until `.DayPicker-Caption` matches the month, then click the `.DayPicker-Day:not(.DayPicker-Day--outside)` with the right day number. Start input is `#startDate`; due is the other `input[placeholder="Select date"]`. New priorities default the due date to the cycle end.
- **Past due dates** trigger an "Are you sure?" modal (`.chakra-modal__content-container`) — click "Confirm" or the date silently stays unchanged.
- **Snapshot UIDs:** scoped `take_snapshot` calls can reuse the same UID for different elements. Prefer `evaluate_script` with explicit selectors, or verify which element a UID points to before `fill_by_uid`.
- **Script timeouts:** `evaluate_script` calls hit a BiDi timeout after roughly 10–20s even with a larger `timeout`, though the script keeps running in the page. Keep each call short (one priority per call), or poll page state with quick calls before acting again — never resubmit blindly, or you'll create duplicates.
- **Roadmap quirks:** the sheet can contain duplicate rows (create once) and typos in titles — flag them to the user before creating.
- Full-page snapshots are truncated before the goals list; use `evaluate_script` / `get_page_text` instead.
