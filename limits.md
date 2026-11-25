# Limits and the guard

Every vault carries its own limits. The bot's creator sets them for the bot's vault; you set them for your copy. They sit inside outer ranges held in ProtocolBounds, readable from `/v1/config` or straight from the contract.

| Limit | Field | Range | Checked when |
| --- | --- | --- | --- |
| Leverage | `maxLeverageBps` | above 0, up to 50,000 (5x) | every order, across all markets together |
| Price band | `priceBandBps` | above 0, up to 50 (0.5%) | every order that opens or grows a position |
| Drawdown | `maxDrawdownBps` | 500 to 5,000 (5% to 50%) | `checkGuard` |
| Daily loss | `maxDailyLossBps` | 100 to 2,000 (1% to 20%) | `checkGuard` |
| Heartbeat | `heartbeatTimeout` | 3,600 to 86,400 seconds | `checkGuard` |
| Markets | `assetMask` | any subset of the listed markets | every order |
| Order size | `maxOrderNotional` | above 0, in USDC | every order |

Tradable markets today: BTC, ETH, SOL, HYPE, PAXG, XRP, BNB, DOGE, AVAX, LINK, SUI, NEAR, ZEC, AAVE, LTC, ADA, UNI, ENA, TAO and ARB. `/v1/assets` lists them with their HyperCore index.

## Exposure

Before sending an order the vault adds up the gross exposure it would hold across every market. If the total would pass the leverage cap, the order shrinks until it fits. Sizes round toward zero, so a vault trades at most its target.

## The guard

`checkGuard()` can be called by anyone. It trips the vault when:

- equity falls below the guard peak by more than the drawdown limit,
- the day's loss passes the daily limit, or
- the bot has sent neither a target nor a `ping` for longer than the heartbeat.

A tripped vault takes reduce-only orders only. `unwind(asset)` closes a position with reduce-only orders priced at the edge of the exit band (3%), so the close goes through in a fast market; anyone can call it. The owner reviews the vault and resumes it when ready.

Deposits raise the guard peak and the day's starting equity, and withdrawals and fees scale them down in proportion, so moving money in or out leaves the guard where it stood.

[Back to the index](README.md)
