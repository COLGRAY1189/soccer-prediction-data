# Soccer prediction — ratings data

Machine-readable ratings snapshots for a soccer prediction app. Published by a daily
GitHub Action from a private app repo. Nothing here is written by hand.

**This repo holds only JSON.** No database, no archive, no binary, no Git LFS. A publish
that would add anything else is refused by a check in the app repo before the commit is
made.

## Layout

| Path | What it is |
|---|---|
| `latest.json` | The most recent snapshot. Read this first. |
| `snapshots/YYYY-MM-DD.json` | One dated snapshot per day, kept for a rolling **120 days**. |

Older snapshots are deleted. A fixture older than the retention window cannot be
replayed, and the app records that as *"not covered — no snapshot"* rather than guessing.
A gap is meant to look like a gap.

## What a snapshot contains

```jsonc
{
  "generatedAt": "2026-08-28T14:10:00.000Z",
  "modelConfig": { /* every coefficient, floor, calibration knot and league list */ },
  "poisson":      { "mu": 0.3, "gamma": 0.22 },   // 180-day fit, feeds match winner
  "poissonGoals": { "mu": 0.31, "gamma": 0.21 },  // 365-day fit, feeds over 2.5
  "leagueOverRate": { "eng.1": 0.55 },
  "teams": { "359": { "elo": 1833.444, "attack": 0.591891, "...": 0 } },
  "build": { "refreshed": ["eng.1"], "stale": [], "gateWindow": {}, "seasonsCached": 288 }
}
```

Roughly 128 KB per snapshot.

### `modelConfig` is duplicated in every snapshot, on purpose

It is not redundancy to be factored out. A prediction is a pure function of
`(snapshot, fixture)`, so a snapshot carries the entire model that produced it. That is
what makes a pick from three months ago reproducible today, and what stops a future model
version from quietly rewriting a past prediction. It costs about 7 KB a day.

### `teams` is keyed by ESPN competitor id, never by name

Names are not identities. In the training data "OH Leuven" appears under two different
ids. Any join, map or comparison on a display name is a bug.

### Float precision

Team ratings are published rounded — elo to 3 decimal places, every rate and log-rate to
6 — which cuts the file by 28%.

Measured across 3.6 million fixtures, that moves a published probability by at most
**4.6e-5**. The only pick it can move is one already sitting within 4.5e-7 of a decision
floor; for context, the model's own accuracy interval is ±2.5 percentage points.

### `build.stale`

A league whose current season could not be refreshed is frozen at its last good ratings
and listed here. It is still published; it is not pretended to be fresh. A league more
than 7 days stale fails the build outright rather than shipping week-old ratings dressed
up as current ones.

## Licence and warranty

Model output, published as a record of what was predicted before kickoff. No odds, no
expected value, no staking advice. Use at your own risk.
