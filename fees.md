# Fees

| Fee | Who pays | Rate | Split |
| --- | --- | --- | --- |
| Copy price | The buyer, once per copy | Set by the creator, 1 USDC or more | 95% creator · 5% protocol |
| Performance fee | Each copy | Set by the creator, 0% to 30% of profit above the high-water mark | 80% creator · 20% protocol |
| Hosted engine | The bot's own vault, for bots run by the Isometry engine | 10% of profit above the high-water mark | Protocol |

Bots run from their creator's own server pay no engine fee on their own vault.

## High-water mark

A vault's high-water mark starts at its first deposit and rises with every deposit. A performance fee counts only equity above it, and after each charge the mark moves up to the equity left in the vault. A copy that loses and recovers pays nothing until it is back above its previous best.

## When fees are charged

`crystallise()` charges the fee due and can be called by anyone, at most once every 30 days per vault. A withdrawal also charges the fee due on the share withdrawn, in the same step.

Rounding favours the payer: the fee rounds down, the protocol's share rounds down, and the creator receives the remainder, so the two parts always add up to the fee charged.

## Changing a fee

A creator can lower the performance fee at any moment. A higher fee takes effect seven days after it is set, and only after each copy has been charged at the old rate, so a raise applies to new profit only.

## Where the protocol's share goes

The protocol's share of copy prices and performance fees is sent to the FeeCollector contract.

[Back to the index](README.md)
