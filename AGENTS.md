# Build conventions for this repository (token-radar-01 entry)

This file is the project instruction file for the coding agent building this entry.

You are building an entry for the make.fun "token-radar-01" bounty: a token launch
radar over a sealed DefiLlama snapshot. Read these rules before every change.

## Hard constraints (the gate refuses entries that break them)

- Node 22, TypeScript run directly by node, standard library ONLY. No npm
  packages, no network access at runtime, no external files read at runtime
  besides the snapshot input.
- Deterministic output: the program runs twice and both runs must write byte
  for byte the same files. Never use Date.now, Math.random, or iteration over
  unordered structures when producing output. Sort explicitly.
- The dashboard (dashboard/) renders with NO network: never put an http:// or
  https:// address in an HTML attribute, a CSS file, a style body, or a JS
  import/export/require/dynamic-import. URLs are allowed in visible text and
  comments only.
- Keep the whole dashboard source (index.html + app.css + app.js) under 60 KB.
- board.json rows: 100 to 4000 rows. Every row: slug (one of the snapshot's
  protocols, unique), name (non-empty), rank (1..N, each once), implied_value_usd
  (positive integer, never rising as rank ascends), farm_steps (3 to 12 strings,
  each 20 to 600 characters after trimming).
- Every slug in the snapshot's watchlist must appear among the rows.

## The model (lib/model.ts)

- Implied value must follow from the snapshot's own figures only: TVL, 7-day
  TVL change, days listed (age), category, chains. Never special-case a
  protocol by name; one function scores every row.
- Peer multiple: for each category, take the tokenized protocols of that
  category (mcap_usd present), compute mcap/TVL for each, winsorize the ratios
  to the category's 10th-90th percentile band, and use the median. Blend the
  category median with the global median by peer count (full weight at 8+
  peers). Categories with no tokenized peers use the global median.
- implied = TVL x peerMultiple x growthFactor x maturityFactor, rounded to a
  whole dollar, at least 1. growthFactor from change_7d clamped to [-50, 100]:
  1 + (clamped/100) x 0.5. maturityFactor: 0.7 + 0.3 x min(age_days, 1460)/1460;
  unknown age counts as 365 days.
- The board ranks pre-token protocols only (no gecko_id, no symbol, no mcap).
  Cover ALL of them, not just the watchlist.
- Farm steps come from per-category playbooks you write in the model file:
  concrete actions with dollar costs and time commitments, parameterised by
  the protocol's name, chains and TVL tier. No slogans, no step repeated
  verbatim across every row: at least the first step names the protocol and
  its main chain.

## Style

- Plain TypeScript, small functions, comments that explain the why.
- The dashboard is hand-written HTML/CSS/JS, no framework. Dark radar theme:
  near-black background, one green accent, one amber accent, system font stack
  with a monospace for numbers.
