---
layout: default
title: Things to ask
nav_order: 3
---

# Things to ask

Copy-paste queries grouped by what you're trying to find out. Each comes with a one-line description of what kind of answer ColdTrace gives back.

This page is for finding something to try.

> **Note on dates.** The example queries below reference specific dates from the data ColdTrace was tested against. If you're running against a fresh install with different loaded data, replace the dates with ones that exist in your system. The shape of the query is what matters.

---

## Track a specific aircraft

When you have an ICAO hex, registration, or callsign and want to see where that aircraft went.

| Query | What you get back |
|---|---|
| *"Show me everywhere aircraft `71e260` flew between April 26 and April 29 2026."* | Full multi-day track for the named aircraft, plotted on the map. |
| *"What was the flight path of aircraft `71e282` on April 27 2026?"* | Single-day track. Faster, narrower. |
| *"Where did aircraft `857ce6` fly between April 26 and April 29 2026?"* | Same shape — named ICAO, time range, full track on map. |

**What this exercises:** the system jumps straight to the aircraft you named — no broad region scan, no scoring. If the aircraft has data in the time window, you'll see every recorded position.

---

## Scan a region for aircraft activity

When you don't know which aircraft you're looking for — you want to see what was flying in an area.

| Query | What you get back |
|---|---|
| *"Were there any unusual flights near Japan between April 28 and April 30 2026?"* | Top ~20 flagged aircraft from the region, scored on ten anomaly signals (transponder gaps, unusual altitudes, etc.). The map shows their tracks. |
| *"What aircraft were over the Strait of Hormuz on 2026-04-26 between 09:00 and 12:00 UTC?"* | Same region-scan path, scoped to a tighter bbox + time window. |

**What this exercises:** the system infers a bounding box from the place name, scans all aircraft in that box during the window, and ranks them by anomaly signal. The report calls out the most interesting ones.

---

## Investigate a specific vessel

When you have an IMO, MMSI, or vessel name and want its history.

| Query | What you get back |
|---|---|
| *"Investigate vessel `9309227` — port visits, sanctions history, and aircraft proximity over the last 90 days."* | Per-vessel report: port visits, current/expired sanctions listings, ownership notes, any aircraft that came near during events. |
| *"Show me anything unusual about FU YUAN YU 7630 in the last 30 days."* | Name-resolution path. Same downstream shape — the system resolves the name to an IMO via the Global Fishing Watch database. |
| *"Investigate vessel SEEKER 8."* | Open-ended vessel investigation; the system picks a sensible default time window. |

**What this exercises:** maritime specialists run in parallel — one walks the GFW event history (port visits, loitering, gaps), another walks the sanctions chain (active listings, past listings, ownership hints). Both flag the same vessel = strong signal.

---

## Scan a region for vessel activity

When you want to find suspicious vessel behavior in a geographic area.

| Query | What you get back |
|---|---|
| *"Show me sanctioned vessel activity in Singapore Strait over the last 90 days, and any aircraft that came close."* | Ranked list of sanctioned vessels with port visits in the region, plus any aviation activity in the same bbox/window. |
| *"Show me vessel activity in the South China Sea in April 2026."* | Pure region scan — port visits and AIS gaps, ranked by signal score. |
| *"Show me port visits and AIS gaps near Iranian ports last month."* | Same shape; the system infers a Persian Gulf bbox from "Iranian ports". |

**What this exercises:** the port-behavior specialist scans GFW events inside the region+window and ranks vessels by a heuristic score (port-visit count, gap count, long stays, sanctioned status). The top ~25 come back with per-vessel summaries.

---

## Cross-domain (aircraft + vessels together)

The questions that are hard to answer with single-source tools. This is where ColdTrace earns its keep.

| Query | What you get back |
|---|---|
| *"Show me aircraft activity over Mumbai between 18.5 and 19.3 north and 72.5 and 73.2 east on 2026-04-26 at 01:00-04:00 UTC, and any vessels with IMO `9332834` in that region."* | Both domains run in parallel. The report includes a Joint Correlations section if any aircraft was close to the named vessel during one of its events. |
| *"Show me aircraft activity over the Strait of Hormuz between 25 and 26 north and 56 and 57 east on 2026-04-26 at 09:00-12:00 UTC, and any vessels with IMO `9309227` in that region."* | Similar shape — Khor Fakkan / GLOBAL DIGNITY (US OFAC SDN). Eleven joint pairings returned in testing. |
| *"Show me sanctioned vessel activity in Singapore Strait over the last 90 days, and any aircraft that came close."* | The Singapore demo's anchor query. Returns the regional sanctioned-vessel scan + an honest "no joint air-sea proximity correlations identified" when no aircraft happened to loiter near the vessels. |

**What this exercises:** all relevant specialists fire (aviation + maritime + sanctions). A separate deterministic correlator (running in Python alongside the LLM agents) computes spatio-temporal proximity pairs and feeds them to the report. The report cites the closest pair if any, or says "no proximity correlations" honestly if there aren't any.

**Tunable knobs** for cross-domain queries (in the Options panel):
- *Joint radius* — default 10 nm. Lower for "actually hovering close"; higher for "in the same area."
- *Joint time window* — default 60 min. Lower for near-coincident timing.
- *Maximum aircraft altitude* — default unlimited. Set to ~5,000 ft to exclude high-altitude overflights.

---

## Live → Investigate handoff

You can also kick off an investigation from the **LIVE** tab without typing anything. Pick a region on the LIVE map, watch a vessel or aircraft, then click its marker — the popup offers a **🔍 Investigate** button that flips into the **INVEST** tab with the composer pre-filled.

For an aircraft, the pre-filled query looks like:
> *"Investigate aircraft `<callsign>` — flights over the last 90 days and any vessels that came close."*

For a vessel:
> *"Investigate vessel `<token>` — port visits, sanctions history, and aircraft proximity over the last 90 days."*

You can send the pre-filled question as-is, or edit it to broaden / narrow before submitting. This is how most "I'm watching something happening *right now*, what's its history?" questions get into the system.

More on this flow in [live-mode.md](live-mode.md).

---

## Follow-ups

Once a turn has finished, follow-ups in the same conversation reuse the evidence already gathered. Some shapes that work well:

**Zoom in on something the report flagged.**

| Follow-up | What it does |
|---|---|
| *"Tell me more about ICAO `300000`."* | Narrows the report to one entity. No fresh data pull — much faster than turn 1. |
| *"What does the AIS gap pattern look like in detail?"* | Drills into one finding from the prior turn. |

**Compare to a different time window.**

| Follow-up | What it does |
|---|---|
| *"Compare aircraft activity over Hormuz between 2026-04-26 09:00–12:00 UTC and 2026-04-19 09:00–12:00 UTC."* | Same area, different time — specialists run again for the new window, but the report calls out what changed vs the first turn. |
| *"How does this week's port activity compare to last week's?"* | Same shape — keep the place, swap the time. |

**Push back on a conclusion.**

| Follow-up | What it does |
|---|---|
| *"What if the 39-minute transponder blackout is a sensor coverage gap, not deliberate suppression?"* | The system re-evaluates the prior conclusion with your alternative explanation in scope. No new data pull — it's a reasoning step over the existing findings. |
| *"Could that aircraft's racetrack pattern be authorised refuelling, not surveillance?"* | Same idea — alternative-explanation re-synthesis. |

**Just re-run the same question.**

If you submit the same question twice in the same conversation, the system detects it and reuses the previous turn's plan + findings rather than re-running everything. The second answer is faster and identical — useful when you want a stable answer to share without LLM-call drift.

---

## When the answer is "nothing"

A useful answer is one you can trust. When the agents don't find a connection, ColdTrace says so explicitly:

> *"No joint air-sea proximity correlations were identified."*
> *"Vessel not found in GFW data — may be unsanctioned, unresolved, or outside coverage."*
> *"No aviation detections recorded in the search bounding box during the entire period."*

These aren't error messages — they're reports of the system's actual conclusion. A query that runs to completion with zero findings is a successful query.

---

## What's next

- [getting-started.md](getting-started.md) — first-time tour of the app.
- [reading-results.md](reading-results.md) — what each piece of the report means.
- [how-it-works.md](how-it-works.md) — what happens between your question and the answer.
