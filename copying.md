# Copying a bot

## Buying a copy

`CopyMarket.buyCopy(botId, limits, depositAmount)` takes the copy price and the first deposit in one transaction, pays 95% of the price to the bot's creator and 5% to the protocol, clones a vault for you and deposits into it.

A bot can be copied when its creator has listed it and its own vault holds at least 10 USDC. Each bot takes up to 1,000 copies. A creator cannot buy a copy of their own bot.

`/v1/bots/:id/quote` answers whether a bot takes copies now, its price, its performance fee and a suggested first deposit.

## How targets reach you

A bot sends a target per market as a share of equity, for example `targetBps = -6750` for a short worth 67.5% of equity. BotHub stores it with a nonce, and `propagate` carries it to the copies in batches and to the bot's own vault last. Anyone may call `propagate`; for every listed or hosted bot, the Isometry keeper calls it for any target still on its way 30 seconds after it was sent. A target that has not reached a vault within 10 minutes expires for every vault, the bot's own included.

Your vault then:

1. multiplies the target by your size setting,
2. converts it to a position size from your equity and the oracle price,
3. holds gross exposure across all markets under your leverage cap and the order under your order size cap,
4. prices the order inside your price band, or inside the wider exit band for an order that only reduces,
5. sends it to HyperCore as an immediate-or-cancel limit order.

If any step stops the order, the vault records the reason (`ApplySkipped`) and the other vaults in the batch carry on.

## Sizing a copy

`setScale(bps)` multiplies every target the bot sends, from 1,000 (10%) to 30,000 (300%). It applies before the leverage cap, so a larger scale still stays inside your own limits.

## Leaving

`pause()` stops following for a while; `detach()` leaves the bot for good. Either way you can close positions and withdraw.

[Back to the index](README.md)
