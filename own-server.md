# Running a bot from your own server

Hosted bots are built and run on [isometry.me](https://isometry.me). This page is for a bot whose logic runs on your own machine and sends targets with its own key.

## 1. Register

On isometry.me choose **Create**, then **Your own server**, and give the address of the operator key your server will sign with. Under the hood this calls:

```solidity
BotHub.registerBot(
    Kind.External,   // 1
    operator,        // the key your server signs with
    perfFeeBps,      // 0 to 3000
    copyPrice,       // USDC with 6 decimals, at least 1_000000
    limits           // the limits of the bot's own vault
);
```

The call mints the bot NFT to you and clones the bot's own vault with you as its owner. Fund that vault with `deposit` like any other.

`setOperator(botId, newOperator)` swaps the operator key at once, so a leaked key is replaced in one transaction.

## 2. Send targets

```solidity
BotHub.setTarget(botId, asset, targetBps, limitPx);
```

| Argument | Meaning |
| --- | --- |
| `asset` | HyperCore asset index, from `/v1/assets` |
| `targetBps` | Share of equity to hold: `10000` is 1x long, `-5000` is 0.5x short, `0` is flat. At most the leverage cap in either direction |
| `limitPx` | Worst price you accept, as price times 1e8. Each vault clamps it into its own price band |

Each call returns a nonce and counts as a heartbeat. Targets reach the copies first and your own vault last; see [Copying a bot](copying.md).

A minimal sender with viem:

```ts
import { createWalletClient, http } from 'viem';
import { privateKeyToAccount } from 'viem/accounts';
import { botHubAbi, hyperEvm } from 'isometry-interfaces';

const BOT_ID = 5n; // the id registerBot returned
const account = privateKeyToAccount(process.env.OPERATOR_KEY as `0x${string}`);
const wallet = createWalletClient({ account, transport: http('https://rpc.hyperliquid.xyz/evm') });

// Hold BTC long at 1.5x, accept up to 60,500 USD.
await wallet.writeContract({
  chain: null,
  address: hyperEvm.contracts.botHub,
  abi: botHubAbi,
  functionName: 'setTarget',
  args: [BOT_ID, 0, 15000, 60_500n * 10n ** 8n],
});
```

## 3. Stay alive

A vault's guard trips when the bot is silent for longer than that vault's heartbeat limit, which can be as short as one hour. When there is no new target to send, call `ping(botId)` from the operator key at least that often.

## 4. List for copies

`list(botId)` opens the bot for copies once its own vault holds at least 10 USDC. `setCopyPrice`, `setPerfFee` and `unlist` adjust the offer; `retire` ends the bot for good, and its copies stop receiving targets.

Your bot's record on Isometry is its own vault's history, and the same numbers are served by `/v1/bots/:id/performance`.

[Back to the index](README.md)
