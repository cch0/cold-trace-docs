---
layout: default
title: How it works
nav_order: 4
---

# How investigations work

Plain-English walkthrough of what happens between the moment you press **Send** and the moment a report appears on your screen.

No code, no class names, no internal jargon. If you want the engineering view — node graph, model IDs, schema details — see [agent.md](https://github.com/cch0/cold-trace/blob/main/docs/agent.md).

---

## The five steps

Every investigation goes through the same five steps. You can see them happen on screen, in order, while the progress indicator turns green.

### 1. You ask a question

You type in plain English. There's no template, no required syntax. *"Investigate this vessel,"* *"What's flying over here?"*, *"Find me the sanctioned tankers in this strait"* — all work.

The question can be specific (mentions an ICAO, IMO, vessel name) or open-ended (just names a region and a time window). Both shapes are fine; the system figures out what you meant.

### 2. The system understands what you're asking

Before pulling any data, ColdTrace reads your question and extracts what it needs to know:

- **What entities are involved?** Aircraft ICAOs, vessel IMOs or names, registrations.
- **What geographic area?** From a region name ("Japan", "Persian Gulf") or explicit coordinates.
- **What time window?** From phrases like *"last 90 days"*, *"between April 26 and April 30"*, or *"yesterday"*.
- **What's the intent?** A scan? A specific entity investigation? A follow-up to an earlier question?

This step is fast (a few seconds). If you submit a follow-up question, this is also where ColdTrace works out whether you're talking about something from a prior turn (*"tell me more about that vessel"*) — and if so, which one.

### 3. It picks the right specialists and runs them in parallel

ColdTrace is organised as a team. Each specialist owns one data source:

- An **ADS-B specialist** for historical aircraft tracks.
- A **vessels specialist** for vessel event history (port visits, loitering, encounters, transponder gaps) from the Global Fishing Watch database.
- A **sanctions specialist** for current and past sanctions listings on vessels and entities.
- An **ownership specialist** for registered operators and beneficial-owner notes.
- A **port-behavior specialist** for regional scans of port-visit patterns.

Based on what your question is about, a **planner** decides which specialists to dispatch. Aviation-only question? Just the ADS-B specialist runs. Vessel-focused? The vessels and sanctions specialists. Cross-domain? All of them.

The dispatched specialists run **in parallel** — each one independently pulling its own data, running its own scoring logic, and producing its own findings. They don't share state mid-investigation. They each get the question and report back.

This is the slowest step. A broad regional scan can take a minute or two; a narrow entity-focused query is faster.

### 4. It cross-checks the findings

After all the specialists finish, two things happen before the report is written:

**A deterministic cross-domain check.** If your question is cross-domain (aircraft *and* vessels), a separate routine runs over both sets of findings and looks for spatial + temporal proximity pairs — aircraft that were close to a flagged vessel during one of its events. This isn't done by an agent; it's a straightforward geometry calculation. The results feed into the report as the **Joint Correlations** section.

**A timeline pre-pass.** Findings from different specialists get woven together into one chronological event list — so the report can describe what happened in order across air, sea, and sanctions sources rather than treating each source as its own silo.

### 5. It writes the report — and a critic checks it

Finally, a **synthesizer** reads all the findings (plus the joint correlations and the timeline) and writes one structured report:

- A short summary at the top.
- A section per source — what the ADS-B specialist found, what the vessels specialist found, etc.
- A **convergence** list — entities flagged by more than one source. (When two specialists independently point at the same vessel, that's a much stronger signal than either alone.)
- A **joint correlations** section, if any aircraft were close to any vessel during the window.
- A timeline of events in chronological order.
- A conclusion and a list of recommended actions.

After the synthesizer writes the report, a **critic** re-reads it against a set of consistency checks. If two specialists disagreed about the same entity — say the maritime specialist rated a vessel as "routine commercial" while the sanctions specialist rated it "high risk" — the critic catches the disagreement and surfaces it in the report rather than letting the synthesizer silently pick a winner.

If the critic flags problems, the synthesizer gets one revision pass to address them. Then the report is final and gets shown to you.

---

## Why this shape

The design has three deliberate properties that come up if you use the system for a while.

**Specialists, not one big model.** Each specialist has its own tools and its own definition of "anomalous" for its domain. That separation is what lets the system disagree with itself when the data disagrees with itself. A single model trying to be everything-at-once tends to smooth disagreements into a coherent-sounding narrative; this design surfaces them as findings.

**Calibrated answers.** When the agents don't find a connection, the report says so. The synthesizer is reading structured findings from independent agents — it doesn't have a single narrative to defend, so it's free to report a quiet finding as a quiet finding. A query that runs to completion with zero findings is a successful query.

**Conversation memory.** Once a turn finishes, its findings are persisted with the conversation. Follow-ups in the same conversation reuse those findings rather than re-running everything — so your second question is much faster than your first.

---

## What you see while it runs

The card in the middle column of the **INVEST** tab shows progress in real time:

- A short status line ("investigating…", "writing report…")
- A progress overlay listing each specialist as it starts and finishes
- A timer
- A **Cancel** button if you want to stop

If you switch to the **DEBUG** tab while a question is in flight, you'll see the same progress in more detail — the parsed plan, a Gantt-style bar showing which specialist took how long, and the raw events streaming in. The Debug tab is most useful when something feels slow or surprising — it answers *"what was the system actually doing for those two minutes?"*

---

## What's next

- [getting-started.md](getting-started.md) — first-time tour.
- [reading-results.md](reading-results.md) — what each piece of the report means.
- [example-queries.md](example-queries.md) — things to ask.
- [agent.md](https://github.com/cch0/cold-trace/blob/main/docs/agent.md) — the engineering view (node graph, model IDs, schemas).
