# Per-attendee resolver prompt

Use this prompt when dispatching one `general-purpose` Agent per attendee. The Agent has WebSearch and WebFetch.

## Resolver prompt template

```
Research the academic {NAME} ({AFFILIATION_HINT or "affiliation unknown"}).
I need the following, in this order:

1. AFFILIATION & POSITION. Their current institution and rank (Asst/Assoc/Full Prof, postdoc, PhD student, research economist, etc.).

2. RESEARCH INTERESTS. One or two sentences on the broad themes they work on, in their own framing if possible.

3. MOST CITED PAPER. Find their highest-cited paper from Google Scholar. Report:
   - Title, venue, year, citation count
   - 2-4 line plain-English summary of the contribution

4. LATEST WORKING PAPER. The most recent working paper they have posted (SSRN, NBER, personal page) — published or unpublished. If only published papers are findable, report the most recent and note that. Report:
   - Title, date, venue (e.g., "SSRN WP, Apr 2026" or "Forthcoming, AER")
   - 2-4 line plain-English summary

Search strategy:
- Start with: "{NAME}" {affiliation hint} Google Scholar
- If Scholar isn't directly accessible, look for their personal/university page, CV, or SSRN author page
- For latest WP: SSRN author page, NBER author page, RePEc/IDEAS, their personal site

Output format (Markdown, no preamble):

### {NAME} — {Affiliation, position}
**Interests.** ...

**Most cited.** *{Title}* ({venue}, {year}, {N} cites). {2-4 lines}

**Latest WP.** *{Title}* ({date}, {venue}). {2-4 lines}

**Sources.** Scholar profile / SSRN / NBER / personal page URLs used.

If you cannot confidently identify the person (multiple matches, none clearly active in academia), return exactly:
[NEEDS DISAMBIGUATION] {brief note on the candidates you found}

Do NOT write "connection to user's work" — that's added later by the calling agent who knows the user's profile.
```

## Notes for the calling agent

- Substitute `{NAME}` and `{AFFILIATION_HINT}` per dispatch.
- Cap each agent at ~6 search/fetch round-trips; if it's still uncertain after that, return [NEEDS DISAMBIGUATION].
- The resolver should NOT write the "Connection to your work" line — that requires the user profile, which only the orchestrator agent reads.
