---
layout: default
title: Getting started
nav_order: 2
---

# Getting started

A short tour for someone who just opened ColdTrace for the first time. By the end of this page you'll know what the four tabs are for, how to ask your first question, and how to read the answer.

If you're looking for setup instructions (Docker, environment variables, bringing the stack up), that's in the [runbook](https://github.com/cch0/cold-trace/blob/main/docs/runbook.md). This page is for using the app once it's running.

---

## What ColdTrace is

ColdTrace is a multi-source intelligence tool for aviation and maritime questions. You type a question in plain English; the system pulls evidence from aircraft tracking feeds, vessel-tracking feeds, sanctions records, and ownership records — in parallel — and writes you back a structured report.

The point of the design is to make **cross-source** questions cheap. Questions like *"what aircraft were near this vessel during its port visit?"* used to mean opening ten browser tabs. Here they're one query.

## The four tabs

The left rail has four tabs. Each one is for a different shape of work.

| Tab | What it's for |
|---|---|
| **INVEST** | Ask questions. This is where investigations happen. Chat-style interface — type a question, get a report, ask follow-ups. The default tab when you open the app. |
| **LIVE** | Real-time map. See aircraft and vessels moving in real time over a region you pick. Watch specific entities, watch trails build, jump into an investigation by clicking a target. |
| **DATA** | Backfill historical data. If your operator hasn't loaded historical aircraft tracks for a date you care about, you can request them here. Most analysts won't need this often. |
| **DEBUG** | Look inside the last investigation. Shows the parsed plan and a timing breakdown of which agents ran when. Useful for understanding why a question took the time it took. |

You'll spend almost all your time on **INVEST** and **LIVE**.

## Your first investigation

1. Open the app. You land on the **INVEST** tab.
2. The screen has three columns: a list of past conversations on the left, a main thread in the middle, and a map on the right. All three are empty on first run.
3. At the bottom of the middle column is a text box: *"Ask a follow-up…"*. Click it and type a question.
4. Try one of these — they each work against the data ColdTrace ships with:
    - *"Were there any unusual flights near Japan last week?"*
    - *"Show me sanctioned vessel activity in Singapore Strait over the last 90 days."*
    - *"Investigate vessel SEEKER 8 — port visits, sanctions history, and aircraft proximity over the last 90 days."*
5. Press **⌘↵** (or click **Send**).
6. The middle column fills with a card: the parsed question, then a progress indicator while the agents work. A broad regional question typically takes a couple of minutes; a follow-up is faster.
7. When it finishes, you'll see a structured report — headline, key findings, recommended actions, and a list of source-specific sections you can expand. The map on the right fills with whatever aircraft or vessel positions the report references.

For what each piece of the report means, see [reading-results.md](reading-results.md).

For more examples of questions to try, see [example-queries.md](example-queries.md).

## Follow-up questions

Once a turn has finished, you can ask a follow-up in the same conversation. ColdTrace remembers what it already found and reuses it.

This means follow-ups are typically much faster than the first question, because the system doesn't have to re-run all the underlying queries — it just re-reads the evidence already gathered.

Some natural follow-ups:

- *"Tell me more about Tanjung Pelepas."* (zooms in on a place mentioned in the previous answer)
- *"What about the same window last week?"* (same location, new time)
- *"What if the aircraft were authorised refuelling flights, not surveillance?"* (alternative explanation)

For more on how this works, see [how-it-works.md](how-it-works.md).

## Live mode

The **LIVE** tab is a real-time view of aircraft and vessels in a region you choose. You can:

- Pick a region preset (Japan, Singapore, Gulf of Mexico, etc.) or draw your own bounding box on the map
- Add specific aircraft or vessels to a watch list — those get a coloured trail behind them as they move
- Click any marker to see its details, or hit the **🔍 Investigate** button in its popup to flip into the **INVEST** tab with a question about that entity already drafted for you

LIVE is what you use when something is *happening right now* (an active incident, a specific aircraft in flight, a vessel approaching a port). INVEST is what you use for *historical* questions ("what happened during last week's port visit?").

For the full LIVE-mode reference, see [live-mode.md](live-mode.md).

## The Options panel

In the top-right of the **INVEST** tab there's an **⚙ Options** button. It opens a panel where you can:

- Pick which model ColdTrace uses for the heavier reasoning step (Sonnet by default; switch to a faster/cheaper Haiku for quick scans, or a beefier model for harder problems)
- Pick which model writes the report (same idea, separate setting)
- Adjust how many findings the report includes by default

Most of the time the defaults are fine. The model picker is useful when you're either rushed (drop to Haiku) or stuck (try a stronger model).

## Glossary

These terms come up throughout ColdTrace. You don't need to know all of them to use the app, but seeing them once helps.

| Term | What it means |
|---|---|
| **ADS-B** | Automatic Dependent Surveillance–Broadcast. The signal aircraft transmit with their position, altitude, and identity. Most commercial flights have it. |
| **AIS** | Automatic Identification System. The maritime equivalent of ADS-B — ships broadcasting their position, identity, and course. |
| **ICAO** | A unique 6-character hex code assigned to each aircraft (sometimes called the "ICAO 24-bit hex" or "Mode S address"). Example: `71e260`. |
| **MMSI** | Maritime Mobile Service Identity. A 9-digit identifier broadcast by ships over AIS. Example: `228459900`. |
| **IMO** | International Maritime Organization number. A 7-digit identifier permanently attached to a vessel's hull, kept across name and flag changes. Example: `9309227`. |
| **Flag** | The country a vessel is registered under. Shown as a two-letter country code (e.g. `pa` for Panama). |
| **bbox** | Bounding box — a rectangular geographic area defined by its SW and NE corners. ColdTrace uses bboxes to scope investigations to a region. |
| **OSINT** | Open-Source Intelligence. Intelligence work done with publicly available data (ADS-B feeds, AIS feeds, sanctions lists) rather than classified sources. |
| **OFAC** | The US Office of Foreign Assets Control. The body that maintains the US sanctions list (the "SDN list"). |
| **EO 13902** | Executive Order 13902 — the US authority for sanctioning Iran's petroleum sector. Often referenced when a vessel is sanctioned for Iranian-oil-related activity. |
| **GFW** | Global Fishing Watch. A non-profit that publishes curated vessel event data (port visits, loitering, encounters, transponder gaps) derived from AIS plus satellite imagery. ColdTrace uses GFW for historical maritime context. |
| **Port visit** | A discrete event when a vessel arrives at a port, stays for some time, then departs. GFW publishes these as time-bounded events with a port ID. |
| **Transponder gap** (or *AIS gap*) | A period when a vessel's AIS broadcast went dark. Sometimes routine (equipment failure, coverage), sometimes a sanctions-evasion signal. |
| **Joint correlation** | A pairing between an aircraft and a vessel that were close in space and time — typically used to flag aircraft passing near a flagged vessel during one of its events. |
| **Convergence** | When two or more independent sources flag the same entity. Strong signal — both the aviation and sanctions specialists pointing at the same aircraft is much more interesting than either alone. |
| **Specialist** | One of ColdTrace's agents. Each specialist owns one data source (ADS-B specialist, vessels specialist, sanctions specialist, ownership specialist, port-behavior specialist). |

---

## What's next

- [example-queries.md](example-queries.md) — copy-paste questions to try, grouped by intent.
- [reading-results.md](reading-results.md) — what each piece of the report means.
- [how-it-works.md](how-it-works.md) — plain-English explanation of what happens between your question and the answer.
- [live-mode.md](live-mode.md) — real-time monitoring details.
- [use-cases.md](https://github.com/cch0/cold-trace/blob/main/docs/use-cases.md) — full feature inventory.
