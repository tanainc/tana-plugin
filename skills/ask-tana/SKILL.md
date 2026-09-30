---
name: ask-tana
description: "Answer a quick factual question from the team's Tana meetings and docs in one or two sentences, with the source: what was decided, when something was last discussed, who owns something, which date, number or name was agreed, what was shown on screen. Use for short questions like 'what did we decide on pricing?', 'when did we last talk about the Android app?', 'who owns onboarding?', 'what deadline did we agree with Acme?', 'what did Mads show in the demo?', 'did we pick Postgres or SQLite?'. For the full history of a topic, use find-prior-art instead."
---

# Ask Tana

A small question deserves a small answer: the fact, where it came from, and a link. No essay.

## Steps

1. **Search narrowly, in one call.** Use the specific words the answer would contain.
   ```json
   { "queries": ["pricing decision", "pricing"],
     "targets": [{ "target": "text" }, { "target": "transcript" }, { "target": "event" }],
     "limit": 15 }
   ```
   (`searchItems`). If the wording is uncertain, also run
   `semanticSearchItems({ query: "what we decided about pricing", limit: 10 })`.
   - "When did we last...": search with one target only, `{ "target": "event", "eventStartTimeMax": "<now, local>", "eventTimeZone": "<tz>" }`,
     plus `"sortOptions": [{ "field": "startTime", "direction": "desc" }]` (startTime sorting needs the event target alone).
   - "Who owns...": look for a doc or task on the topic (`searchItems` text target; results carry
     `assignedTo`), and resolve profile URIs to names with `listOrgMembers`.

2. **Read only the best source.**
   - A doc: `readItems({ ids: [id], task: "<the question>" })`.
   - A meeting: find the passage with `readFullTranscript` (narrow with `startSec` / `endSec` once you
     know roughly where), then **check the screen at that moment**:
     `readScreenShareScreenshots({ id: callUri, startSec, endSec })`. Numbers, dates, names and
     choices are often on the shared screen (a pricing table, a roadmap, a chosen option), and the
     screenshot is more exact than what was said. When the answer is a number, date or name on that
     frame, read it off the pixels: `readScreenshotImages({ id: callUri, cids: [cid] })`.
     See the `tana-meeting` skill for the pairing.
   - A pasted Tana link: read exactly that item (the `tana-meeting` skill shows how).

3. **Check it is still true.** If a later meeting or doc changed the answer, the latest one wins.
   A quick `searchItems` sorted by recency on the same keywords is enough.

4. **Answer** with the template. Get the link with `getShareLink({ ids: [...] })`.

## Output

```
**$49 per seat per month, annual only.** Decided in Pricing sync, Tue 22 Sep (at 18:40), proposed
by Hilde and agreed by Mads; the pricing table was on screen. <link>
```

When the answer changed over time:
```
**Postgres.** Decided in Architecture review, 14 Sep (at 05:10) <link>.
(An earlier sync on 2 Sep had leaned towards SQLite.)
```

## Rules

- One or two sentences. Bold the answer.
- Always name the source: meeting or doc, date, and who, plus mm:ss for meetings.
- Say "decided" only if it was decided. Otherwise: "discussed, not decided".
- Not found? "I couldn't find that in Tana." Then one line on what you searched.
- If the user wants more, offer the `find-prior-art` history in one line.
