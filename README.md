# How QRS Works 

QRS is a Flask + SQLite web app that helps you reason about road safety around a location without slurping up personal data. You give it coordinates and a little context (vehicle type, destination). It then:

1. **Locates the area.**
   It tries an LLM-based reverse geocode to name the nearest place; if that’s unavailable, it falls back to a local database (Geonames) and a distance function to pick the closest town/city.

2. **Builds a probability picture.**
   A small quantum-inspired routine (via PennyLane) turns a few live system signals (CPU/RAM as stand-ins for “environment noise”) into a probability vector. Think of it like rolling multiple weighted dice that represent “possible road-state futures.” It’s not magic—just a compact way to get a distribution you can reason about.

3. **Synthesizes a concise report.**
   The app uses a templated prompt (if an API key is present) to draft an ultra-brief hazard summary: debris, collision potential, weather interaction, and suggested actions. If the LLM is unavailable, QRS still stores the location context and your inputs for later review.

4. **Stores with privacy in mind.**
   Coordinates, text, and metrics are encrypted at rest (AES-GCM). Sessions are rotated regularly. Reports are rate-limited, and old records are securely overwritten and purged (not just “deleted”), then vacuumed.

5. **Shows results safely.**
   Markdown is sanitized, CSRF is enabled, and registration can be invite-only. Admin seeding is environment-driven so you don’t need to click through a “create admin” screen.

# Probabilistic Intent: spotting hazards by likelihood, not certainties

Most “detectors” try to say *what is there* right now. QRS leans into **intent**—what the road is *likely trying to do to you* given incomplete information. It mixes:

* **Spatial priors:** Where you are (urban vs rural) changes odds for debris, lane closures, or pedestrian risk.
* **Context cues:** Vehicle type and destination shape which hazards matter (a motorcycle cares about different debris than a semi).
* **Distributional scan:** The quantum-inspired step yields a probability spread over “hazard states.” It’s deliberately noisy; we want ranges, not a false sense of certainty.
* **Language constraints:** The report generator is instructed to be terse, avoid certain trigger words unless warranted, and to focus on actionable recommendations. That keeps the narrative aligned with risk signals, not drama.

“Probabilistic intent” here means QRS models *tendencies* rather than claims. If multiple weak signals nudge toward “loose gravel” or “lane blockage ahead,” QRS elevates that scenario’s weight and suggests cautious, low-regret actions (slow, reroute, or watch speed over turns).

# Why You Should Care (≈500 words total)

Road risk is messy. You rarely have perfect, sensor-grade truth, yet you still need decisions *now*. QRS embraces that reality. It doesn’t pretend to be a police scanner, a traffic feed, or a lidar stack; it’s a **lightweight decision aid** that combines location context, compact probability modeling, and careful prompting to produce **short, practical guidance**.

The upside:

* **Lower cognitive load.** You get a 1-screen summary: likely hazards, why they matter for *your* vehicle, and what to do next. No scrolling through social feeds or wading through overconfident maps.
* **Fewer false alarms.** By treating outputs as probabilities, QRS avoids hard calls when evidence is thin. Uncertain? It nudges a cautious posture rather than shouting “danger.”
* **Privacy by default.** Coordinates and text are encrypted at rest; the app never needs your identity beyond login, and cleanup uses multi-pass overwrite (not a perfunctory delete).
* **Portable & cheap.** It’s a simple Flask app with SQLite. You can host it on Replit or a \$5/mo VM and still keep strong security practices (CSRF, invite codes, secret rotation).
* **Extensible.** Want to plug in weather APIs, municipal feeds, or dashcam telemetry later? The pipeline already separates “context → distribution → summary,” so you can add stronger signals without rewriting the app.

Who is this for?

* **Solo drivers & small fleets** that want a quick “sanity check” before committing to a route—especially in unfamiliar areas.
* **Civic hackers / researchers** exploring risk communication: how do we give humans the *right* amount of caution without crying wolf?
* **Builders** who want a template for privacy-respecting, probability-aware apps: encrypted storage, safe inputs, zero dynamic SQL, and short, useful outputs.

In short: QRS helps you make **reasonable, low-regret driving choices** under uncertainty. It won’t replace real sensors or official advisories—but it can bridge the everyday gap between *no signal* and *overconfident signal*, nudging you toward safer behavior when it matters.
