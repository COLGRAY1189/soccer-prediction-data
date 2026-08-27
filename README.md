# soccer-prediction-ratings

Data-only feed for a personal soccer match-prediction app. **No application code lives here** — this
repository exists so an installed app can fetch fresh team ratings without a server, and so its
predictions can be reproduced exactly after the fact.

Everything here is generated. Nothing in it is edited by hand.

## Layout

```
latest.json              the most recent snapshot
snapshots/YYYY-MM-DD.json  dated snapshots, retained for 120 days
```

`latest.json` appears after the first build runs; until then this repository is empty by design.

## What a snapshot contains

A snapshot is a single JSON object holding team ratings **and the exact model configuration that
produced them**:

| Field | Meaning |
|---|---|
| `generatedAt` | ISO timestamp the snapshot was built |
| `modelConfig` | Model version, Elo parameters, blend weight, over-2.5 weights, calibration knots, confidence floors, and the per-market league lists |
| `poisson` | Global scoring level and home-advantage factor, log-scale |
| `leagueOverRate` | Recent over-2.5 rate per league |
| `teams` | Per-team ratings, **keyed by numeric team id** |

Teams are keyed by id and never by display name. Names get re-spelled between reads, and in this
dataset one display name (`"OH Leuven"`) maps to two genuinely different clubs — a name-keyed join
would silently merge them.

## Why the config travels with the ratings

A prediction is a pure function of `(snapshot, fixture)`. Because each snapshot carries its own
coefficients, calibration and floors, a pick made weeks ago can be re-derived exactly, using only
information that provably predates the match — even after the app ships a newer model. Without that,
a model update would silently rewrite the app's own past predictions, and its accuracy record would
become fiction.

That is also why dated snapshots are kept rather than only the latest one.

## Retention

Snapshots are pruned to a rolling **120 days**. A fixture older than that window cannot be replayed,
and the app records it as *"not covered — no snapshot"* rather than guessing. A gap is shown as a
gap.

These are small text files (roughly 100 KB a day). **Nothing binary, archived or database-shaped is
ever committed here** — a previous project of mine was lost after a daily job committed a large
binary through Git LFS until it exhausted the storage allowance, and LFS storage cannot be reclaimed
without deleting the repository.

## Provenance

Ratings are derived from publicly available match results, on a personal, non-commercial basis. The
files here are computed aggregates — team strength estimates and fitted model coefficients — not a
copy or redistribution of any source dataset.

## Not a prediction service

These numbers are inputs to a personal hobby app. They are not advice of any kind, there is no
uptime or accuracy guarantee, and the format may change without notice. The model is deliberately
market-free: no odds are used anywhere in its construction.
