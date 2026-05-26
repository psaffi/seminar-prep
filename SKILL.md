---
name: seminar-prep
description: Generates a pre-meeting briefing for an academic seminar visit. Given a schedule (paste, list, or file path) with names of people you'll meet, produces a Markdown + HTML briefing with each person's affiliation, most-cited paper, latest working paper, research interests, and a connection-to-your-work line. Use when the user mentions a seminar trip, a meeting schedule, or asks to prep for talks/visits.
---

# Seminar Prep Briefing

Produces a one-stop briefing on the people the user is meeting at a seminar day. For each attendee, look up affiliation, most-cited paper, latest working paper, research interests, and write a short connection-to-the-user's-work line. Output both Markdown (editable) and a self-contained HTML page (printable).

## When to use

- "Prep for my seminar trip to X"
- "I'm meeting these people tomorrow, give me briefings"
- User pastes a meeting agenda with names
- User points to a calendar export or .txt schedule

## Inputs

Accept any of:
- **Paste** — agenda text with names interleaved with times/events (parse names out)
- **List** — bulleted or comma-separated names, optionally with affiliations
- **File path** — `.ics`, `.md`, `.txt`, or `.csv` with names

If a name is ambiguous and no affiliation is given, ask once (batch) before starting research.

## Workflow

1. **Parse** the input. Extract `{name, affiliation?, time?, event?}` per attendee. Drop non-person items (Arrival, Snack, Seminar, Lunch unless a specific host is named).
2. **Read user profile.** Load `user-profile.md` from this skill folder — it lists the user's research themes. This feeds the "connection to your work" line.
3. **Disambiguation pass.** If any name is ambiguous (multiple academics with that name and no affiliation supplied), ask the user once with a batched AskUserQuestion before dispatching research.
4. **Research in parallel.** For each attendee, dispatch a `general-purpose` Agent (via the Task/Agent tool) with the resolver prompt from `references/search-strategy.md`. For ≤3 attendees, do searches inline instead.
5. **Synthesize per-person card.** Each card contains:
   - Name, affiliation, position
   - 1-2 line research interests
   - Most-cited paper: title, venue, year, citations — then 2-4 line summary
   - Latest working paper: title, date, venue — then 2-4 line summary
   - Connection to your work (1-2 lines)
6. **Compose outputs.**
   - `briefing.md` — Markdown with schedule table at top, cards in meeting order
   - `briefing.html` — self-contained HTML via the template in `templates/briefing.html`
   - `briefing.pdf` — render the HTML via Edge headless: `msedge --headless=new --disable-gpu --no-pdf-header-footer --print-to-pdf="<path>\briefing.pdf" "file:///<path>/briefing.html"`. Edge is at `C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe` or `C:\Program Files\Microsoft\Edge\Application\msedge.exe` on this machine. Skip if Edge is unavailable.
7. **Save** to `~/Dropbox/Claude/seminar_briefing/YYYY-MM-DD_{host-slug}/`. If host institution is unclear, ask the user once or fall back to `YYYY-MM-DD_seminar`. The parent folder `~/Dropbox/Claude/seminar_briefing/` is a git repository — after saving, `cd` in and `git add -A && git commit -m "Add briefing: {host} {date}"`.
8. **Report** file paths and offer to open the HTML or PDF.

## Output structure (Markdown)

```markdown
# Seminar Day Briefing — {Host institution}, {Date}

## Schedule
| Time | Meeting |
|------|---------|
| 10:00 | Hendrik Döpper |
| ...   | ...           |

## Attendees

### 1. Hendrik Döpper — {Affiliation, position}
**Research interests.** ...

**Most cited paper.** *{Title}* ({venue}, {year}, {citations} cites). {2-4 lines}

**Latest working paper.** *{Title}* ({date}, {venue}). {2-4 lines}

**Connection to your work.** {1-2 lines}

---
```

## Output location

All briefings live under `~/Dropbox/Claude/seminar_briefing/` (a git repo). One subfolder per trip: `YYYY-MM-DD_{host-slug}/`. After writing files, stage and commit them with a `Add briefing: {host} {date}` message so the trip is versioned.

## Rules

- Always cite **two papers** per person: one most-cited, one latest working paper. If you cannot find a recent working paper, fall back to most-recent-published and label it as such.
- 2-4 lines per paper summary. Not 5. Not 1.
- "Connection to your work" must be **specific**, not generic. Reference the user's themes from `user-profile.md` (equity lending, CDS, short-selling supply/demand, NAIC, IV designs, etc.) and find a concrete overlap or contrast.
- If a person cannot be confidently identified, write a brief card with a `[NEEDS USER INPUT]` flag and proceed with the rest.
- Never fabricate citations. If you can't find a paper, say so.
- Use `WebSearch` first (broad), then `WebFetch` on the top hit (deep). Never paste raw search-result blobs into the briefing.
- HTML must be self-contained (inline CSS), print-friendly, no external assets.

## Search source priority

Per attendee, try in order until you have what you need:
1. Google Scholar profile (search: `"{name}" "{affiliation hint}" google scholar`)
2. SSRN author page
3. NBER author page
4. RePEc / IDEAS author page
5. Personal university page
6. Recent CV PDF if linked

See `references/search-strategy.md` for the resolver agent prompt.
