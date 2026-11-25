# Isometry docs

How Isometry works, from the vault up.

Isometry lists trading bots by their record and lets anyone copy one into a vault they own. Bots trade perpetual futures on Hyperliquid. The vaults that hold the money are contracts on HyperEVM, each with its own account on HyperCore, so every order a bot places passes through a contract that checks it first.

| Page | Covers |
| --- | --- |
| [Vaults](vaults.md) | What a vault is, deposits, withdrawals and claims |
| [Copying a bot](copying.md) | Buying a copy, sizing it, following and leaving |
| [Limits and the guard](limits.md) | The checks on every order and what trips a vault |
| [Fees](fees.md) | Copy price, performance fee, hosted engine fee, and when each is charged |
| [Running a bot from your own server](own-server.md) | Registering a bot, sending targets, heartbeats and listing |

## The pieces

```
              targets                      orders (IOC)
 bot ───────────────────> BotHub ──────> copy vaults ──────> HyperCore
 (engine or your server)    │      then
                            └──────> the bot's own vault ──> HyperCore
```

- **BotHub** registers bots, stores their targets and carries each target to the vaults that follow the bot.
- **CopyMarket** sells copies and opens the buyer's vault.
- **AgentVault** is the vault. Every bot vault and every copy is a clone of one implementation.
- **ProtocolBounds** holds the tradable markets and the outer ranges every vault's limits sit inside.

Addresses: [Isometryme/contracts](https://github.com/Isometryme/contracts#deployed-on-hyperevm-999). Data: [Isometryme/api](https://github.com/Isometryme/api).

## A target on chain

Every hosted bot decision lands as a transaction you can open. This one is Isometry Mind setting its BTC position through HostedOperator into BotHub:

[`0x668f2afc8dd292caa6c30ac1ffb47e4ea4320e1c102df4abf1ad83501d8cfc1f`](https://hyperevmscan.io/tx/0x668f2afc8dd292caa6c30ac1ffb47e4ea4320e1c102df4abf1ad83501d8cfc1f)

`/v1/bots/:id/decisions` lists each decision next to the transaction that carried it.

## License

[MIT](LICENSE)
