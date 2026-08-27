# How to verify this German ledger (for a skeptic)

The claim: every decision in `Data/receipts.jsonl` was committed BEFORE the
DE-LU prices it is judged against were published, and every P&L in
`Data/ledger.jsonl` follows mechanically from those receipts and public
prices. Don't trust us — check it.

## 1. Timing (receipts predate the auction)

The German day-ahead auction publishes delivery-day T prices ~12:40 CET on
T-1. A receipt for target T is honest only if it existed before that.

- **git-attested:** every tick commits and pushes `Data/`. Compare a
  receipt's `committed_at` (UTC, in the file) with the push time of the
  commit that introduced it: `git log --follow --format='%H %cI' --
  Data/receipts.jsonl`. The commits are authored on the always-on server.
- **Bitcoin-anchored (OpenTimestamps) from 2026-08-27:** each tick now
  stamps `Data/ots/<date>.txt` (a sha256 manifest of receipts + ledger) and
  a `<date>.txt.ots` proof, upgraded to Bitcoin anchoring weekly. Verify:
  `sha256sum Data/receipts.jsonl` against the manifest, then
  `ots verify Data/ots/<date>.txt.ots`. Evidence tiers, stated plainly: the
  earlier DE days (2026-08-24 → 2026-08-26) are git-attested only — OTS
  anchoring cannot be backdated (that is the point of the mechanism), so it
  begins the day it was added.

## 2. Arithmetic (P&L follows from receipts + public prices)

DE-LU day-ahead prices are public: Bundesnetzagentur | SMARD.de (CC BY 4.0).
For any settled day:

    pnl = sum(price[h] for h in sell_hours) * 1.0 MW * 0.85
        - sum(price[h] for h in buy_hours) * 1.0 MW
        - 0.5 EUR/MWh * (2 * 1.0 + 2 * 0.85)

Oracle = same formula over the day's 2 cheapest / 2 dearest hours;
capture = pnl / oracle_pnl. Exact code: `src/esios_paper/loop.py` (pure
functions, `uv run pytest`). Prices are keyed by Europe/Berlin local hour;
SMARD serves 15-minute values which are aggregated to hourly means upstream.

## 3. No cherry-picking

Both audit files are append-only: every receipt has a settlement or a
documented missed day. Losing days stay identically. Strategy changes
require a new pre-registered strategy id; old receipts stand. Stop
conditions and the comparison bar are pre-registered in GOVERNANCE.md.

## 4. What this ledger does NOT claim

Paper trading — no market participation, execution assumed at the clearing
price (realistic for a 1 MW price-taker). Fees cover the exchange only (no
grid charges, taxes, aggregator margin), so absolute EUR is an upper bound;
relative metrics (capture, tau, deltas) are robust. Hourly frame; the market
clears at 15-minute granularity. This is a SIGNAL record, not a real-money
execution record.
