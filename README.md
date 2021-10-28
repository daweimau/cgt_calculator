# CGT calculator

A silly helper notebook for calculating capital gains on stocks.

## Use case

Trading stocks is fairly straightforward to track manually. There are many reports available.

When stocks are sold, capital gains (or losses) become assessable.

Calculating the assessable gains can quickly become difficult, especially when holdings were acquired incrementally over time. Sales rarely occur in the same increments as those in which the stocks were originally bought. This makes tracing the gains difficult.

# For serious users

This is a common task! There's nothing fancy in this repo - it was created mostly as a fun exercise. A great service that does this calc, and much more is available at https://www.sharesight.com/ . They have a free offering.

### Methodology notes

In Australia there are also various acceptable tax methodologies.

This is a script which takes a FIFO ('first in, first out') approach. It reads in your trades, and produces a readable report of capital gains (including 12 month 50% CGT discounted assessable gains). 

# Example scenario / breakdown of this script's approach to FIFO calcs

1. Start with a 'raw' (transformed, if required) input: a record of historic trades. Note for simplicity this example has just one stock at play (the script works through every separate stock as required)

| type | date        | ticker | u  | ppu  | total |
| ---- | ----------- | ------ | -- | ---- | ----- |
| buy  | 1 Jan 2020  | NAB    | 3  | 1.5  | 4.5   |
| buy  | 2 Feb 2021  | NAB    | 2  | 1.5  | 3     |
| sell | 3 Mar 2021  | NAB    | 1  | 1.75 | 1.75  |
| sell | 4 Apr 2021  | NAB    | 3  | 2.8  | 8.4   |

2. Sort these by date, oldest at top.

3. Explode these into single row per unit.

| type | date       | ticker | u | ppu  | total |
| ---- | ---------- | ------ | - | ---- | ----- |
| buy  | 1 Jan 2020 | NAB    | 1 | 1.5  | 1.5   |
| buy  | 1 Jan 2020 | NAB    | 1 | 1.5  | 1.5   |
| buy  | 1 Jan 2020 | NAB    | 1 | 1.5  | 1.5   |
| buy  | 2 Feb 2021 | NAB    | 1 | 1.5  | 1.5   |
| buy  | 2 Feb 2021 | NAB    | 1 | 1.5  | 1.5   |
| sell | 3 Mar 2021 | NAB    | 1 | 1.75 | 1.75  |
| sell | 4 Apr 2021 | NAB    | 1 | 2.8  | 2.8   |
| sell | 4 Apr 2021 | NAB    | 1 | 2.8  | 2.8   |
| sell | 4 Apr 2021 | NAB    | 1 | 2.8  | 2.8   |

4. Separate the buys and sells into separate dataframes
5. Discard excess length on the buys table (from the bottom, i.e. discard the still-unsold, most recently purchased holdings)

| buys |||||| |sells||||||
| ---- | ---------- | ------ | --- | --- | ----- | --- | ---- | ---------- | ------ | --- | ---- | ----- |
| type | date       | ticker | u   | ppu | total |     | type | date       | ticker | u   | ppu  | total |
| buy  | 1 Jan 2020 | NAB    | 1   | 1.5 | 1.5   |     | sell | 3 Mar 2021 | NAB    | 1   | 1.75 | 1.75  |
| buy  | 1 Jan 2020 | NAB    | 1   | 1.5 | 1.5   |     | sell | 4 Apr 2021 | NAB    | 1   | 2.8  | 2.8   |
| buy  | 1 Jan 2020 | NAB    | 1   | 1.5 | 1.5   |     | sell | 4 Apr 2021 | NAB    | 1   | 2.8  | 2.8   |
| buy  | 2 Feb 2021 | NAB    | 1   | 1.5 | 1.5   |     | sell | 4 Apr 2021 | NAB    | 1   | 2.8  | 2.8   |
| ~~buy~~  | ~~2 Feb 2021~~ | ~~NAB~~    | ~~1~~   | ~~1.5~~ | ~~1.5~~   |     |      |            |        |     |      |       |


6. Comparing each unit sell vs buy to evaluate gain, hold duration, and discount availability

| buy  |||||| sells |||||||||
| ---- | ---------- | ------ | --- | --- | ----- | ----- | ---------- | ------ | --- | ---- | ----- | -------------- | --------------------- | -------------------- | 
| type | date       | ticker | u   | ppu | total | type  | date       | ticker | u   | ppu  | total | **gain\_loss** | **12month\_discount** | **assessable\_gain** |
| buy  | 1 Jan 2020 | NAB    | 1   | 1.5 | 1.5   | sell  | 3 Mar 2021 | NAB    | 1   | 1.75 | 1.75  | **0.25**       | **TRUE**              | **0.125**            |
| buy  | 1 Jan 2020 | NAB    | 1   | 1.5 | 1.5   | sell  | 4 Apr 2021 | NAB    | 1   | 2.8  | 2.8   | **1.3**        | **TRUE**              | **0.65**             |
| buy  | 1 Jan 2020 | NAB    | 1   | 1.5 | 1.5   | sell  | 4 Apr 2021 | NAB    | 1   | 2.8  | 2.8   | **1.3**        | **TRUE**              | **0.65**             |
| buy  | 2 Feb 2021 | NAB    | 1   | 1.5 | 1.5   | sell  | 4 Apr 2021 | NAB    | 1   | 2.8  | 2.8   | **1.3**        | **FALSE**             | **1.3**              |

7. Aggregate/pivot the results for a human-readable summary

| Discount applies | Ticker | Date sold  | Units sold | Assessable gain |
| ---------------- | ------ | ---------- | ---------- | --------------- |
| FALSE            | NAB    | 4 Apr 2021 | 1          | 1.3             |
| TRUE             | NAB    | 3 Mar 2021 | 1          | 0.125           |
|                  |        | 4 Apr 2021 | 2          | 1.3             |
| **Total** ||| **3**          | **1.425**           |

