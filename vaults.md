# Vaults

A vault is a contract on HyperEVM with its own account on HyperCore. It follows one bot, and it belongs to one owner: the bot's creator for the bot's own vault, the buyer for a copy.

Positions, equity and fills live on HyperCore. The vault reads them through HyperCore's read precompiles and places orders through CoreWriter, so the money stays in the vault and every order is sent by the vault itself.

## States

| State | Meaning |
| --- | --- |
| Active | Follows every target from the bot |
| Paused | The owner paused it; it ignores new targets until resumed |
| Tripped | The guard tripped; it takes reduce-only orders until the owner resumes it |
| Detached | The owner left the bot; the vault keeps its funds and stops following |

## Deposits

`deposit(amount)` moves USDC from the owner's wallet to the vault's HyperCore account through Circle's CoreDepositWallet. The amount raises the vault's high-water mark and its guard peak by the same amount, so a deposit counts as capital and profit starts above it.

HyperCore credits a deposit a few seconds after the transaction. Anyone can then call `settle()` to book what arrived. A deposit that opens a new HyperCore account arrives up to 1 USDC lighter, the activation charge HyperCore takes once per account. If HyperCore has still not credited a deposit after 24 hours, the owner can cancel it.

## Withdrawals

`withdraw(amount)` asks HyperCore to send USDC back to the vault on HyperEVM, up to the amount HyperCore reports as withdrawable. Any performance fee due on the withdrawn share is taken in the same step. Once the transfer lands, `settle()` books it and `claim()` pays the owner, the bot creator and the protocol what each is owed.

The vault waits a short moment (20 HyperEVM confirmations) after its last order or transfer before it starts a withdrawal, so it always sizes from settled numbers.

## Owner controls

| Function | Effect |
| --- | --- |
| `setLimits(limits)` | Changes the vault's limits at once, inside the outer ranges |
| `pause()` / `resume()` | Stops and restarts following |
| `closePosition(asset)` | Closes one position with reduce-only orders |
| `detach()` | Leaves the bot for good |
| `sync(asset)` | Applies the bot's latest target for one market now, useful after joining late or resuming |
| `setScale(bps)` | On a copy, scales every target from 10% to 300% |

[Back to the index](README.md)
