# seminar-prep

A Claude Code skill that turns a seminar-day schedule into a pre-meeting briefing. For every person on the schedule it pulls affiliation, research interests, most-cited paper, latest working paper, and writes a connection-to-your-work line. Outputs Markdown, self-contained HTML, and a print-rendered PDF.

## Install

```bash
git clone git@github.com:psaffi/seminar-prep.git ~/.claude/skills/seminar-prep
```

The folder must live at `~/.claude/skills/seminar-prep/` for Claude Code to discover it. After cloning, edit `user-profile.md` to match your research themes — that file feeds the "connection to your work" line in every briefing.

## Files

| File | Purpose |
|---|---|
| `SKILL.md` | Frontmatter + workflow Claude follows when the skill fires |
| `user-profile.md` | Your research themes (edit freely; the skill rereads each run) |
| `references/search-strategy.md` | Per-attendee resolver prompt dispatched to a sub-agent per person |
| `templates/briefing.html` | Self-contained HTML template (inline CSS, print-ready) |

## Use

In a Claude Code session, invoke any of:

- `/seminar-prep` and paste the schedule
- Natural language: "prep for my seminar at Cambridge Tuesday" + schedule
- Point at a file: `/seminar-prep ~/Downloads/agenda.txt`

The skill parses names, asks for disambiguation if needed, dispatches one research sub-agent per attendee in parallel, and writes the briefing under `~/Dropbox/Claude/seminar_briefing/YYYY-MM-DD_{host-slug}/`.

## Output

Each trip produces:
- `briefing.md` — editable source of truth
- `briefing.html` — self-contained, print-ready
- `briefing.pdf` — rendered via Edge headless print

## Tuning

Edit `user-profile.md` to add themes, methods, or co-author names. The skill leans on this file to write the "connection to your work" line — the more specific your profile, the sharper the conversation hooks.
