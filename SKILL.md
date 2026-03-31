---
name: akshare-online-alpha
description: Execute Wyckoff-style A-share analysis with full system capabilities: current Beijing-time/session judgment, online multi-source OHLCV fetching with fallback, CSV and chart-image multimodal parsing, and portfolio action inference from holdings(symbol+cost+qty), cash, and optional candidate symbol. Use when users want auditable single-stock or portfolio decisions (switch/add/reduce/hold/exit), strict non-trading-session downgrade behavior, and optional post-session Python chart rendering under fixed prompt constraints.
---

# Akshare Online Alpha

Run this skill as a deterministic orchestration, not as free-form chat.

## Input Protocol

Accept any combination of:
- Stock symbol(s), including single or multiple symbols.
- `holdings`: `[symbol+cost+qty, ...]`.
- `cash`: available cash amount.
- `candidate`: optional non-holding symbol.
- CSV file(s), image(s), text constraints/goals.

Infer scenario automatically:
- `holdings` non-empty + `candidate`: run rotation comparison + per-holding actions.
- `holdings` non-empty + no `candidate`: run per-holding add/reduce/hold/exit.
- `holdings` empty + `cash`: run empty-position cash deployment.
- No portfolio fields: run symbol analysis flow.

Do not require users to explicitly say "switch/add/reduce/empty-position."

## Full Capability Orchestration (Required Order)

1. Parse and normalize inputs.
- Normalize symbols to exchange-qualified format when possible.
- Parse holdings into `{symbol, cost, qty}`.
- Preserve raw user inputs for output audit.

2. Acquire current time via system/tool first.
- Fetch current timestamp from tool/system.
- Convert to `Asia/Shanghai`.
- Print `当前北京时间：YYYY-MM-DD HH:MM（UTC+8）`.

3. Decide trading availability with authoritative calendar checks.
- Judge weekday and continuous-auction windows: `09:30-11:30`, `13:00-15:00` (Beijing time).
- Query authoritative trading calendar when holiday/adjusted-workday uncertainty exists.
- If not tradable, downgrade to post-market review + next-session plan + T+1-safe order strategy.

4. Fetch online data with source fallback.
- Follow `rules/source-fallbacks.md` strictly for each symbol.
- Perform schema and row-count validation before accepting a source.
- Log fallback attempts and final source per symbol.

5. Integrate CSV/image modalities when provided.
- Use CSV as supplemental historical structure input and reconcile with fetched data.
- Treat chart images as micro-structure evidence; explicitly acknowledge image reception.
- Continue analysis if one modality fails and state exact failure cause.

6. Run Wyckoff structural analysis first.
- Analyze latest available window (target 500 trading days) with `MA50`/`MA200`.
- Identify only evidenced phases/events (SC/ST/Spring/LPS/SOS/UTAD).
- Use event-date news search only for validation context, never as primary trade logic.

7. Produce portfolio decisions after structure analysis.
- For each holding, output one explicit action: `add / reduce / hold / exit`.
- If candidate exists, compare against structurally weakest current holding and decide `switch / partial switch / hold`.
- If holdings are empty and cash exists, output staged cash deployment suggestion.

8. Render plots only when session rules allow.
- Skip plotting during tradable intraday windows.
- When plotting is allowed, enforce all constraints in `rules/alpha-system-prompt.md`.

9. Apply capability degrade policy.
- Never fabricate OHLCV rows, event timestamps, or trading status.
- If all sources fail for a symbol, mark `data_unavailable` and continue others.
- If valid rows < 30, report `insufficient structure depth` and avoid hard phase labels.
- If image parsing fails, explain reason and continue CSV/text/online path.

For detailed capability-routing policy, read `rules/system-capability-playbook.md`.

## Fixed Output Contract

Always output in this order:
1. `当前北京时间：YYYY-MM-DD HH:MM（UTC+8）`
2. Trading verdict:
- `当前是否可盘中交易：是/否`
- If no: `当前不可盘中交易（原因：...）`
3. Data audit table per symbol:
- `symbol`
- `source_used`
- `rows_kept`
- `window_end_date`
- `fallback_count`
4. Wyckoff analysis result:
- Current cycle background and phase (only what is evidenced).
- Key events (SC/ST/Spring/LPS/SOS/UTAD if present) with concise rationale.
- Action boundaries respecting T+1 and current session status.
5. Portfolio action section (portfolio flow only):
- Holdings snapshot from provided `cost/qty/cash`.
- Per-holding action suggestions (`add / reduce / hold / exit`) with reasoning.
- Candidate vs weakest holding comparison (if candidate is provided).
- Empty-position cash action suggestion (if holdings are empty).
- Final action summary in Wyckoff tone.
6. Plotting section (only when allowed by session rules):
- Python code and/or generated chart result with Chinese annotation constraints.

## Hard Constraints

- Do not change the fixed prompt wording unless explicitly requested.
- Do not fabricate missing OHLCV rows.
- Do not ignore image input if image is parseable.
- Do not use opaque white text boxes in chart annotations.
- If fetching data requires running Python scripts, run them only in a sandboxed environment.
- Prefer direct web/API fetch first; use Python scripts only when needed for fallback, parsing, or normalization.

## Resources

- `rules/alpha-system-prompt.md`: fixed role and hard rules.
- `rules/source-fallbacks.md`: online source switching policy.
- `rules/system-capability-playbook.md`: full system capability routing and degrade policy.
