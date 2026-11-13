# eth-pnl-fixed — AIProve bounty entry

Donchian 4-hour range breakout on the frozen ETH/USDT 1-minute dataset
(`bars.csv`, 1,440 bars, 24h window ending 2026-11-13 12:17 UTC).

## Strategy

Causal, no lookahead: a decision made at the close of bar `i` is executed at
the **open** of bar `i+1`.

- **Channel:** highest high / lowest low of the previous **240 bars** (4h).
  Enter LONG on a close above the channel high, SHORT on a close below the
  channel low.
- **Risk:** initial stop = entry ± **3.0 × ATR(14)**. Exit on the stop, or
  when price closes back through the opposite channel edge.
- **Size:** 1× notional ($10,000), long / short / flat only. No leverage, no
  pyramiding.
- **Why this shape:** 1-minute return stdev is ~11 bp while a round trip
  costs ~20 bp, so trading 1-minute noise is dead on arrival. The only
  durable edge here is the 4-hour range expansion: the strategy stays FLAT
  through the 12-hour chop and rides the one genuine trend leg.

## Cost model

The challenge README does not fix exact fees, so the model is stated here
and results are shown across a range:

- Round-trip cost **20 bp** by default = **10 bp per fill**
  (~7.5 bp taker fee + ~2.5 bp slippage). Every fill is charged.
- Sensitivity is reported at **10 / 20 / 30 bp** round-trip.

## Results (`python3 main.py`)

| Round-trip cost | Net PnL | Max drawdown | Closed trades | Win rate |
|---|---|---|---|---|
| 10 bp | +$242 | $218 | 2 | 50% |
| **20 bp** | **+$217** | **$228** | **2** | **50%** |
| 30 bp | +$192 | $238 | 2 | 50% |

Walk-forward split at 20 bp: **first half $0** (zero trades — flat through
the chop, exactly as designed) / **second half +$217**. Gross +$211, costs
$50. Final position: long 1× (held into the end of the sample).

Closed trades: SHORT 2006.03 → 2013.77 (−39 bp, stopped out) and
SHORT 1998.10 → 1948.26 (+249 bp, the trend leg).

## Robustness

- Parameter plateau: lookback 200–360 × ATR mult 3.0–4.0 all net positive
  at every cost level (+$177 to +$242). No magic number.
- Trailing (chandelier) exits were tested and **rejected**: they churn the
  position and turn the strategy negative at 20 bp.
- Mean reversion (Bollinger 20/2) was tested and **rejected**: −$1,626 net
  at 20 bp over 78 trades — expected reversions of a few bp cannot clear
  ~20 bp round-trip costs. Same lesson as our BTC bot work, where mean
  reversion was disabled for exactly this reason.
- EMA-crossover trend variants were tested and **rejected**: they whipsaw in
  the chop and lose in the first half.

## Run

```sh
python3 main.py            # reports 10/20/30 bp
python3 main.py --cost 20  # single cost level
```

Stdlib only, deterministic, no network.
