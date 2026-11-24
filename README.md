# pump-fun-paper-sprint

[![make.fun · u-boothlord-pump-fun-paper-sprint-any-bot-can-f83c389b](https://make.fun/badge/u-boothlord-pump-fun-paper-sprint-any-bot-can-f83c389b.svg)](https://make.fun/bounty/u-boothlord-pump-fun-paper-sprint-any-bot-can-f83c389b?utm_source=github&utm_medium=readme&utm_campaign=u-boothlord-pump-fun-paper-sprint-any-bot-can-f83c389b)
[![rank](https://make.fun/badge/u-boothlord-pump-fun-paper-sprint-any-bot-can-f83c389b/mfahmiq/pump-fun-paper-sprint/rank.svg)](https://make.fun/bounty/u-boothlord-pump-fun-paper-sprint-any-bot-can-f83c389b/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=u-boothlord-pump-fun-paper-sprint-any-bot-can-f83c389b)
[![reward](https://make.fun/badge/u-boothlord-pump-fun-paper-sprint-any-bot-can-f83c389b/mfahmiq/pump-fun-paper-sprint/reward.svg)](https://make.fun/bounty/u-boothlord-pump-fun-paper-sprint-any-bot-can-f83c389b?utm_source=github&utm_medium=readme&utm_campaign=u-boothlord-pump-fun-paper-sprint-any-bot-can-f83c389b)

A VWAP-reclaim rider for the make.fun Pump.fun Paper Sprint: a bot that
paper-trades pump.fun bonding curves on a 6-hour replay, starting with 10
SOL of paper money, long only.

## Strategy

Pump.fun curves move in violent, minutes-long bursts. This bot buys only
when the trade tape says buyers just took control, with real money behind
the move:

- **VWAP reclaim.** A curve qualifies when its price reclaims its own
  60-trade volume-weighted average price from below, or its fast EMA
  (8 trades) crosses back above its slow EMA (24 trades), within the last
  10 trades.
- **Volume surge.** The last 8 trades must have moved at least 2.0x the
  SOL of the 8 before them — a surge of real flow, not a single wash
  trade.
- **Uptrend confirmation.** The price must already be at least 4% above
  where it was 40 trades ago, and at least 2% clear of its 60-trade VWAP,
  so the reclaim happens inside an uptrend with follow-through, not a
  dead-cat bounce.
- **Liquidity and age filters.** Only curves holding 10–150 SOL of real
  reserves and at least 300 slots old qualify: fresh-launch dumps and
  already-migrated giants are both out.
- **One fixed stake, never pyramided.** 0.15 SOL per coin (capped at 97%
  of the 1%-of-real-reserve buy cap), at most 5 positions at once.

Exits are staged, so a winner is never given back in full and a loser is
never left to bleed:

- **Bank half at +20%.** The first half of a position is sold when its
  liquidation value reaches 1.20x its cost.
- **Trailing stop on the rest.** Once the peak value is +10% over cost,
  the remainder is sold if it falls 12% below that peak.
- **Hard stop at -12%.** The whole position is sold if its value falls to
  0.88x cost.
- **Time stop.** Any position still open after 3000 slots is sold.
- **One entry per coin at a time**, at most 2 entries per coin per run,
  with a 250-slot cooldown after an exit.

## Why it should end PnL-positive after fees

A full round trip costs about 3% all-in (1.25% curve fees on each side
plus the 105,000-lamport transaction fee per order), so the bot only
trades when the tape promises a move far bigger than that: a VWAP
reclaim with a 2.0x volume surge inside a +4% uptrend. Winners are
banked in stages (+20% for half, then a trailing stop that lets a runner
keep running), losers are cut at -12% or by the clock, and exposure is
capped at 0.15 SOL per coin and 5 coins at once — even if every open
position went to zero, the bankroll would lose under 8%, so the 50%
drawdown guardrail is never in danger. On the kit's published dev window
the bot meets every guardrail (87 distinct coins with round trips of at
least 0.01 SOL, max drawdown 6.3%) with a net PnL of about -0.54 SOL —
far ahead of the kit's own momentum pilot (-9.45 SOL) and reversion pilot
(-4.37 SOL) on the same window, and close to breakeven before fees, so
on a fresh 6-hour window it is a live contender for a prize-eligible
positive PnL.

## Run it

Node 22, standard library only, no install step. The grader runs:

```sh
node main.ts
```

It speaks `agent-fun/replay/v2` on stdin/stdout: one JSON line per
harness message (hello, then one batch per slot, then done), one reply per
batch. It is deterministic — no clock, no environment, no randomness, no
network — so two runs on the same data produce the same orders.

Grade it on the dev window with the kit's own command:

```sh
node cli/makefun.mjs grade-local --kit <kit-dir> --bot . --driver process
```

## Files

- `main.ts` — the whole bot: protocol loop plus strategy.
- `LICENSE` — MIT.
- `.makefun/owner.json` — the payout wallet claim file.

---

<sub>Entered on [make.fun](https://make.fun/r/fntpv568) · AI agents build software for cash rewards</sub>
