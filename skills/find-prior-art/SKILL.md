---
name: find-prior-art
description: "Search the team's Tana meetings and docs for earlier discussion of a topic: past feedback, bug reports, decisions, objections, customer requests, experiments that were tried. Reads the best hits, including the exact transcript passages and screenshots where the topic came up, and reports what was said, when and by whom, with links. Use when the user asks 'has anyone discussed this before?', 'did we try this already?', 'any feedback on X?', 'what have customers said about Y?', 'have we seen this bug before?', 'what's the history of Z?', or is mid-work and wonders whether the team has been here before."
---

# Find prior art

Someone halfway through a piece of work wants to know whether the team has been here before.
The answer usually sits in a meeting: a customer complaining on a call, a bug someone screen-shared
in standup, a design review that already rejected the idea. This skill searches meetings and docs,
reads the passages where the topic came up, and reports the history with links.

## Steps

1. **Frame the search.** Write 2–4 keyword variants (product names, feature names, error strings,
   customer names, the team's slang) and one plain-language description of the topic.

2. **Search wide, in parallel.**
   - Keywords across docs, transcripts and meetings, batched in one call:
     ```json
     { "queries": ["promo code", "discount field", "coupon"],
       "targets": [{ "target": "text" }, { "target": "transcript" }, { "target": "event" }],
       "limit": 30 }
     ```
     (`searchItems`)
   - Meaning, for what nobody named the same way: `semanticSearchItems({ query: "customers confused about where to enter a discount", limit: 20 })`.
   - If the workspace files things like bugs or feedback as typed docs, narrow with the text target's
     `type` (`"type": ["Bug", "Feedback"]`); `getTypes` lists the names. Skip this if it isn't obvious.

3. **Pick the best 5–8 hits.** Prefer exact matches, decisions over chatter, customers over guesses,
   and recent over old, but keep the oldest relevant hit: it shows how long this has been around.

4. **Read them.**
   - Docs: `readItems({ ids: [...], task: "what was said about <topic>: decisions, feedback, who, when" })`.
   - Meetings (hits of kind event, transcript or call share one id: `tana:event:<id>`, `tana:transcript:<id>`,
     `tana:call:<id>`): `readEvent` for the date and people, then `readFullTranscript` and find the
     passage. Read around it with `startSec` / `endSec`.
   - **Check the screen for that passage:** `readScreenShareScreenshots({ id: "tana:call:<id>", startSec, endSec })`
     over the same window. If the frame shows the bug, the design or the data being discussed, say what it
     showed, and look at the pixels with `readScreenshotImages` when the detail matters.
     See the `tana-meeting` skill for how screenshots and transcript line up.

5. **Get links.** `getShareLink({ ids: [...] })` for every meeting and doc you cite.

6. **Report** (template below), newest first.

## Output

```
Yes, this has come up 4 times since March.

- Tue 29 Sep · Design review (Anna, Mads) · <link>
  At 12:10 Anna showed the checkout sketch with the promo field under the total; Mads: "people
  miss it down there." Decided: move it above the total.
- 14 Aug · Acme onboarding call · <link>
  Customer (Lena, Acme) couldn't find where to add their code; screen showed the cart page at 31:40.
- 2 Jul · Bug: "Promo code ignored on mobile" (doc, Completed) · <link>
- 11 Mar · Pricing sync: tried auto-applied codes, dropped (support load). · <link>

What this means now: moving the field is already decided; auto-applying codes was tried and
dropped in March.
```

## Rules

- Quote briefly and exactly; give the date, the speaker and the mm:ss.
- Say what was on screen when it matters ("the error on screen was...").
- Separate what was decided from what was only said.
- Nothing found? Say so, and list what you searched for, so the user can refine.
- Don't read every hit in full. Read the passages that answer the question.
