# A committed-before-truth ledger for German (DE-LU) day-ahead power

**Live dashboard:** see `index.html` (GitHub Pages) · **Audit it yourself:** [VERIFY.md](VERIFY.md)

The public, auditable record of a paper-trading experiment on German
(DE-LU) day-ahead electricity prices. Every day, BEFORE the German
day-ahead auction publishes tomorrow's prices (~12:40 CET), a set of
pre-registered strategies commits receipts simulating a 1 MW / 2 MWh battery
(buy the 2 cheapest hours, sell the 2 dearest; 85% round-trip; explicit
fees). Settlements against the published prices are appended to a ledger
that is never edited.

This is the same methodology and code as the Spanish ledger
(`spain-dayahead-ledger`), run on a second market — the first evidence the
approach is not Spain-specific.

- **Committed before truth**: a receipt for day T exists only if it was
  recorded before T's prices were published — enforced in code (the leak
  guard) and the clock guard.
- **Append-only, losses included**: missed and losing days stay forever.
- **Pre-registered strategies and comparison bars** (see GOVERNANCE.md).

Data source: **Bundesnetzagentur | SMARD.de**, licensed CC BY 4.0
(https://creativecommons.org/licenses/by/4.0/) — the DE-LU day-ahead
wholesale price; values reshaped to JSON, no content altered.

Evidence-tier note (honest): this German ledger is currently
**git-attested** (every tick commits and pushes `Data/`, so commit push
times bound the receipts) but not yet OpenTimestamps/Bitcoin-anchored like
the Spanish ledger — Bitcoin anchoring for DE is a planned addition. Absolute
EUR is an upper bound (exchange fees only). Reproduce: `uv sync && uv run
pytest`, then follow VERIFY.md.
