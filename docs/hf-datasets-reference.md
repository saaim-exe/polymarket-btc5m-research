# HF dataset quick reference

A plain-language guide to [aliplayer1/polymarket-crypto-updown](https://huggingface.co/datasets/aliplayer1/polymarket-crypto-updown), based on the dataset card and preview inspected during our review. The first table is named **`markets`** (plural). Column names and available values should still be checked when loading a new snapshot.

## The five tables at a glance

| Table | What one row represents | What it tells you |
|---|---|---|
| `markets` | One prediction market | Which event it is, when it ends, and its recorded outcome |
| `prices` | A recorded market-price observation | How the Up/Down contract prices changed over time |
| `ticks` | An individual recorded trade fill | What someone bought or sold, at what price, and how much |
| `spot_prices` | An underlying crypto-price update | What BTC itself was worth, rather than the price of a prediction contract |
| `orderbook` | A recorded best buy/sell quote for one outcome token | The best displayed prices and amounts available on each side |

**Example:** `spot_prices` might show BTC at $70,000 while `prices` shows the Up contract at 0.60. These are different prices. The first is the underlying asset; the second is a prediction-contract price, often used as a rough 60% probability estimate.

## 1. `markets`: the event directory

Start here to select BTC five-minute events and find their labels. Each event has an Up token and a Down token: two separately tradable contracts for its possible outcomes.

| Column | Plain meaning |
|---|---|
| `market_id` | The event's identifier; connects this table to prices and trades |
| `question` | Human-readable event description |
| `crypto` | Asset name, such as `BTC` or `ETH` |
| `timeframe` | Event length, such as `5-minute` |
| `volume` | Recorded market trading volume in USDC; not necessarily the amount known at your prediction time |
| `resolution` | Recorded result: `1` = Up, `0` = Down, `-1` = unknown |
| `start_ts` | Metadata start time in seconds; do not assume this is the start of the five-minute price-comparison window |
| `end_ts` | Event end time in seconds |
| `condition_id` | Identifier for the event's on-chain outcome condition; mainly useful for deeper lookups |
| `up_token_id` | Identifier of the tradable Up contract |
| `down_token_id` | Identifier of the tradable Down contract |
| `fee_rate_bps` | Recorded taker fee rate in basis points; 100 basis points = 1%. `-1` means unknown. This field alone does not specify the full fee calculation. |
| `closed_ts` | Extra field visible in the preview; its exact meaning was not explained in the inspected card. Do not assume it proves when the result became known. |
| `slug` | Extra field visible in the preview, intended as an event name/URL identifier; some preview values were empty |

**Useful variables:** `y = resolution`; `prediction_ts = end_ts - 60`; candidate `window_start_ts = end_ts - 300` for a verified five-minute event. Keep timestamps as timing information, not automatic predictive features. A previous event's result is a possible feature only if it was known when making the current prediction.

## 2. `prices`: prediction-contract price history

This is the simplest starting point for the market-probability baseline. Despite the card's heading referring to OHLC history, its listed columns contain Up/Down price observations, not separate open/high/low/close fields.

| Column | Plain meaning |
|---|---|
| `market_id` | Which event the observation belongs to |
| `crypto`, `timeframe` | Asset and event length |
| `timestamp` | Observation time in seconds |
| `up_price` | Recorded Up-contract price, between 0 and 1 |
| `down_price` | Recorded Down-contract price, between 0 and 1 |
| `volume` | Market volume field; the card does not establish that this is volume accumulated only up to this row's timestamp |
| `question` | Repeated event description |
| `resolution` | Repeated outcome information, documented here as a nullable string; use `markets.resolution` for the model target |

**Useful variables:** `market_prob = up_price` at or before your prediction time; `market_momentum_30s = market_prob_now - market_prob_30s_ago`.

A historical price is not necessarily the price you could buy at. Do not assume Up and Down observations are perfectly synchronized or sum to exactly 1. The table does not promise an observation every second.

## 3. `ticks`: actual recorded trades

A fill means a trade was executed. An order can be filled in parts, so a fill row is not necessarily a whole order or a unique trader.

| Column | Plain meaning |
|---|---|
| `market_id` | Which event was traded |
| `timestamp_ms` | Trade time in milliseconds |
| `token_id` | Which tradable contract was involved |
| `outcome` | Whether that contract is `Up` or `Down` |
| `side` | `BUY` or `SELL`, from the perspective of the trader accepting an available order |
| `price` | Execution price of that contract |
| `size_usdc` | Trade value in USDC; this is money, not the number of shares |
| `tx_hash` | Blockchain transaction identifier; empty for WebSocket records |
| `block_number` | Blockchain block containing the trade; 0 for WebSocket records |
| `log_index` | Position of the event within blockchain logs; useful when identifying records |
| `source` | `onchain` or `websocket`: how the trade was collected |
| `spot_price_usdt` | Binance underlying price attached to the trade |
| `spot_price_ts_ms` | Timestamp of that attached underlying-price observation |

**Useful variables:** `trade_volume_30s` = total USDC traded in the preceding 30 seconds; `trade_count_30s` = number of eligible fills; `up_buy_share_30s` = Up BUY value divided by all Up trade value in that window.

Another option is `up_flow_imbalance_30s = (Up BUY value - Up SELL value) / total Up trade value`. Calculate Down separately. Define what happens when no trades occur. Check for duplicate trades across collection sources before counting or summing them. The attached Binance price is not automatically the official settlement reference.

## 4. `spot_prices`: BTC's own price

This table follows the underlying asset independently of any one prediction market. It has no `market_id`: select the appropriate symbol/source and match observations by time.

| Column | Plain meaning |
|---|---|
| `ts_ms` | Price source's timestamp in milliseconds |
| `symbol` | Asset pair, such as `btcusdt` or `btc/usd`; inspect actual values |
| `price` | Underlying crypto price in USD or USDT, depending on the feed |
| `source` | Feed provider; the card lists `binance` and `chainlink` |

The notebook currently filters `chainlink_proxy`, which differs from the card's source names. Check which values actually exist before filtering. USD and USDT prices, and different providers, should not be silently treated as identical.

**Useful variables:** `btc_return_30s` = percentage or log change over 30 seconds; `btc_return_since_open` = change since the event-window start; `btc_volatility_60s` = how much equally spaced returns varied over the preceding minute.

Use one consistent feed for a return calculation. The source timestamp tells you when the observation is dated, not necessarily when your program received it.

## 5. `orderbook`: displayed buying and selling prices

This table contains the **best bid and ask**, not the entire book of orders at every price. The bid is the best displayed buying price; the ask is the best displayed selling price. To buy immediately you generally look at the ask; to sell immediately you look at the bid.

| Column | Plain meaning |
|---|---|
| `ts_ms` | Time the quote update was received, in milliseconds |
| `market_id` | Which event the quote belongs to |
| `token_id` | Which outcome contract is quoted |
| `outcome` | `Up` or `Down` |
| `best_bid` | Highest displayed buying price |
| `best_ask` | Lowest displayed selling price |
| `best_bid_size` | Shares displayed at the best bid |
| `best_ask_size` | Shares displayed at the best ask |

**Useful variables:** `midpoint = (best_bid + best_ask) / 2`; `spread = best_ask - best_bid`; `book_size_imbalance = (best_bid_size - best_ask_size) / (best_bid_size + best_ask_size)` when the denominator is positive.

Example: bid 0.58 and ask 0.62 give midpoint 0.60 and spread 0.04. The midpoint is a probability proxy; it is not an available buying price. Displayed amounts are shares, unlike tick `size_usdc`. Quotes can change before an order arrives, so a snapshot does not guarantee a fill.

## How to connect them in the notebook

1. Select `BTC` and `5-minute` in `markets`.
2. Use `market_id` to find that event's `prices`, `ticks`, and `orderbook` rows. Use token IDs or `outcome` to distinguish Up from Down.
3. Select the BTC feed in `spot_prices` and match by time.
4. At prediction time t, take observations from at or before t and build trailing features ending at t. Never use later observations to fill missing earlier values.

**Watch the units:** `markets` and `prices` times are in seconds; `ticks`, `spot_prices`, and `orderbook` times are in milliseconds. Convert to a common unit before matching.

For initial V0 work, `markets` + `prices` + `spot_prices` are enough to explore the target, market baseline, BTC returns, and volatility. Add trade and quote features as experiments call for them. See [V0 target and features](v0-target-features.md) for the model-focused companion document.
