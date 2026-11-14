# Token Radar — token-radar-01

The best token launch radar: a program (`main.ts`, Node 22, standard library only)
that reads a sealed DefiLlama snapshot of 758 protocols and writes a web dashboard
ranking every pre-token protocol by what its launch would be worth, each with the
steps to farm it.

## The model (radar-2, peer-multiple)

Implied value follows only from the snapshot's own figures:

```
peer     = median market-cap / TVL of the live-token protocols in the same category
           (at least 3 comparables, else the global median), clamped to [0.05, 1.5]
implied  = round( tvl_usd x peer x (1 + clamp(change_7d, -60, 120) / 100 x 0.6)
                  x (0.7 + min(age_days, 1095) / 1095 x 0.3) )
```

A lending protocol is priced the way the market already prices lending protocols
with tokens; a DEX the way it prices DEXes. No protocol name is special-cased.

## The board

- 178 rows: all 40 watchlist protocols plus every other rankable pre-token protocol
  below the $50M floor — the early radar, so a launch that does not exist yet is
  found early. Wrapped, staked and bridged products of tokens that already trade
  (WBTC, Binance staked ETH, bridges) are not ranked: they are no launch of their own.
  A protocol above the floor that the watchlist leaves out is left out too: the house's
  rule excluded it (its parent protocol, a field the snapshot does not carry), so the
  early radar stops at the floor.
- Live-token protocols are set aside with their market caps shown; their ratios set
  the peer multiples.
- Every row carries a seven-step farm plan derived from its category, chains, TVL
  and age, each step with a cost and a time.

## The dashboard

`dashboard/` is a static page with no network: a top-ten spotlight, a sortable and
filterable board (search, category, chain, watchlist-only), a detail view per
protocol with the model's decomposition and farm plan, a live-token view, and a
model sandbox that re-ranks the board live.

## Run it

```sh
mkdir -p out && MAKEFUN_OUTPUT_DIR=out node main.ts --input ../input/snapshot.json
```

That writes `out/board.json`, `out/data.js`, `out/dashboard/`, `out/screenshots/`
and `out/dossier.json`. Run it twice: both runs write the same bytes.

## Layout

- `main.ts` — the entrypoint: read snapshot, build board, write outputs, dossier
- `lib/model.ts` — the peer-multiple model, the board, the farm plans
- `vendor/makefun-board.ts` — make.fun's dossier library, byte for byte as shipped
- `dashboard/` — `index.html`, `app.css`, `app.js`
- `screenshots/` — the board, a detail view, a filtered view (1440x900)

Implied value is this entry's own claim, never one make.fun makes. "No token"
means the snapshot records no token for the protocol, never that it has none.
