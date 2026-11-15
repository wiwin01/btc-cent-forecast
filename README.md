# btc-cent-forecast — make.fun entry for btc-cent-forecast-01

## What it does
`main.py` reads every case file in the input directory (`$MAKEFUN_INPUT_DIR`,
else `/input`) — 4,320 one-minute closes of BTC (the cbBTC/USDC Orca pool
price) over the 72 hours up to a checkpoint — and writes one forecast per
case to the output directory: the price expected 240 minutes after the
checkpoint, as `{"forecasts":[{"case": <id>, "price": <guess>}]}`.

## The model
The score is mean absolute error, so the right point forecast is the
conditional **median** of the target price. The program estimates it from
the case itself:

1. Take every overlapping 240-minute log return whose start lies in the
   **trailing 36 hours** of the window (1,920 returns).
2. Take their **median** — robust to the momentary jumps a single large
   trade in the pool can cause.
3. Guess = last close × exp(0.85 × median): the median drift, **shrunk 15%
   toward zero** to hedge the overshoot when a trend reverses inside the
   horizon.

Standard library only, no network, fully deterministic, runs in
milliseconds per case.

## Practice results
The kit's six practice cases (p00–p05) are checkpoints 4 hours apart, so
the answer to each of p00–p04 is the last close of the next case's series;
p05's answer is not in the kit. On those five cases:

| model | MAE |
|---|---|
| this entry | **$419.42** (41,942 cents) |
| persistence (house baseline) | $470.55 (47,055 cents) |

A rolling-origin backtest over the six 72-hour windows (264 origins) put
the trailing-36h median family at $448–449 against persistence's $451,
and the official 6-case practice baseline score is 44,521.6 cents. The
final is scored on prices nobody has seen, so practice numbers are a
guide, not a promise.

## How it was built
Made by Muse (Muse Spark, in the Muse app) for the make.fun account
@irawati-469: variants (full-window vs trailing-window medians, trimmed
means, shrinkage levels) were scored in a local lab against the practice
cases and the rolling-origin backtest, and the winner was written into
`main.py`.
