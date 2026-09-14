# V0 classifier: target, features and baselines

This is the feature specification for initial logistic-regression experiments in [v0.ipynb](../notebooks/v0.ipynb), not a claim that these features are already implemented. Start with the existing HF data and a small classifier; more accurate classifiers and a more reliable data stream can follow. Settlement-change verification is background research, not a prerequisite for this initial official-label experiment. The previously discussed August 7 change remains unverified; TWAP (time-weighted) and VWAP (volume-weighted) are different mechanisms, so neither is assumed here.

## Target

For each BTC five-minute market, estimate:

**p_up(t) = P(resolution = 1 | X_t)**

- `y = 1`: official Up resolution.
- `y = 0`: official Down resolution.
- Unknown or missing resolutions are excluded from supervised examples.
- `p_down(t) = 1 - p_up(t)`; a separate Down classifier is unnecessary.

`X_t` contains only information available by the prediction time. Start with one prediction per market at **t = end_ts - 60 seconds**, matching the notebook's first proposed offset. Its 90-second and 120-second offsets are separate candidate experiments. For a five-minute contract, `end_ts - 300 seconds` is the candidate event-window start, not a tenth-minute prediction point. Metadata `start_ts` is not used as the event opening reference.

## Initial features

Let `S(t)` be the latest eligible BTC spot observation at or before t, and `q(t)` the latest eligible Up-token price. Use one consistent spot source and match its actual source/symbol values. All lagged observations are obtained backward in time; missing or stale values must not be filled using future observations.

| Feature | Definition | Why include it? |
|---|---|---|
| Current market probability | `q(t)` | Captures the market's current assessment; essential for testing incremental information beyond the market. Historical `up_price` is a price proxy, not necessarily a bid/ask midpoint. |
| BTC short return | `log(S(t) / S(t - 30s))` | Recent underlying price direction and magnitude. Thirty seconds is a starting candidate, not an optimized window. |
| BTC longer return | `log(S(t) / S(t - 60s))` | A slightly longer view of recent movement; compare against the short-return-only version. |
| BTC movement since window start | `log(S(t) / S(window_start))`, with validated `window_start = end_ts - 300s` | Position relative to the beginning of the event. This is an observed spot-return feature, not a reconstructed official settlement label. |
| Trailing volatility | Standard deviation of equally spaced spot log returns over a trailing window ending at t; initial candidate: 60 seconds | Distinguishes a given price movement in quiet versus volatile conditions. Choose and record a sampling interval supported by the observed data; do not treat irregular ticks as equally spaced. |
| Market probability momentum | `q(t) - q(t - 30s)` | Captures recent changes in market assessment without starting with many correlated lagged price levels. |

Begin with current market probability, one BTC return and trailing volatility. Add opening-relative movement, the second return and market momentum one at a time to see whether they help. Raw BTC price can be retained for inspection, but returns are the preferred initial model inputs because the absolute price level changes across the sample. Scale numeric features using training-period statistics only.

At the fixed 60-second horizon, time remaining is constant and adds no information. It becomes a candidate feature only if a later model combines horizons.

## Optional feature additions

| Feature | Candidate definition | Condition for using it |
|---|---|---|
| Trailing traded volume | Sum of eligible tick `size_usdc` over the preceding 30 or 60 seconds | Use deduplicated trades available by t. Final `markets.volume` and undocumented price-table volume are not safe substitutes for historical volume. |
| Previous market outcome | Official Up/Down label of the immediately preceding BTC five-minute event | Include only if its resolution was available by t. A final dataset label does not establish this. Missing event windows must not be skipped as if adjacent. |
| Outcome streak | Number of consecutive prior Up or Down outcomes, or indicators for the last few outcomes | Same availability and adjacency requirements as the previous-outcome feature. |
| CLOB trade imbalance | For each outcome separately: `(BUY notional - SELL notional) / total notional` over a trailing window | Requires source-overlap/deduplication checks and taker-side interpretation. Define zero-trade handling; missing capture is not zero trading. |
| Quote age | Seconds between t and the selected market/spot observation | Identifies stale input conditions. Use an exclusion rule for unusably old observations rather than expecting the model to repair them. |
| Bid/ask spread | `best_ask - best_bid` for the Up token | Optional when usable BBO data is available; not required for the first notebook model. |

Do not include the current market's resolution, final volume, end-of-window spot price, or any observation after t as a feature. Keep the target column separate from model inputs.

## Baseline comparisons

| Baseline | Predicted Up probability | Purpose |
|---|---|---|
| Coin flip | Always `0.5` | Minimal reference for probabilistic prediction. |
| Training prevalence | Fraction of Up outcomes in the training period | Tests whether apparent performance merely reflects class imbalance; never estimate this from the full dataset. |
| Market implied probability | `q(t)`; optionally a fresh BBO midpoint if available | Main benchmark: does the classifier improve on the market's assessment? |
| Market-only logistic regression | Logistic regression using only `q(t)` | Separates simple recalibration of the market price from gains due to extra features. |
| Spot-only logistic regression | Logistic regression using selected BTC returns and volatility | Tests the predictive contribution of underlying-price information alone. |
| Combined V0 classifier | Logistic regression using `q(t)` plus the initial spot features, then incremental additions | Main experiment: measure what each feature group adds over the market-only comparison. |

A digital-option probability can be explored later as an additional baseline once its payoff assumptions and volatility estimator are specified. It is not needed to start V0, and adding it as a feature would not automatically make the classifier learn residual signal.

Compare baselines on the same held-out chronological examples using Brier score and log loss, with accuracy as a secondary measure. Any calibration or feature selection uses development data only. Better probability scores in this experiment do not by themselves establish executable trading profits.
