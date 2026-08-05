---
name: Fund and settle a Lemon Mini App wallet
description: Move crypto between the user's Lemon Cash wallet and the Mini App wallet
  with deposit and withdraw, and handle the PENDING settlement state correctly.
api: '@lemoncash/mini-app-sdk'
api_docs: https://lemoncash.mintlify.app/functions/deposit
operations: [isLemonWebView, authenticate, deposit, withdraw]
operation_kind: sdk-function
generated: '2026-08-04'
method: generated
source: https://lemoncash.mintlify.app/functions/deposit
---

# Fund and settle a Lemon Mini App wallet

`deposit` pulls funds from the user's Lemon Cash wallet into the Mini App wallet.
`withdraw` pushes them back. Both are on-chain and both are user-confirmed inside the
Lemon Cash app.

## Preconditions

- `authenticate` has already succeeded for this session (see
  `lemon-cash-authenticate-mini-app-user.md`). Both calls act on the wallet that
  authentication returned.
- You know which `chainId` you are on — it is a required parameter on both functions.

## Deposit

```ts
const result = await deposit({
  amount: '100',
  tokenName: 'USDC',
  chainId: ChainId.POLYGON,
});
```

`amount` is a string. `tokenName` is one of the published `TokenName` values — AAVE,
ARB, AVAX, AXS, BNB, BTC, CELO, DAI, ETH, GNO, LINK, OP, PAXG, POL, RIF, UNI, USDC,
USDT, USDS, XDAI.

## Withdraw

```ts
const result = await withdraw({
  amount: '50',
  tokenName: 'USDC',
  chainId: ChainId.POLYGON,
});
```

## Handling the result

Both return the same four-state union:

- `SUCCESS` → `result.data.txHash` is final.
- `PENDING` → `result.data.txHash` exists but the transaction is not yet confirmed.
  **Do not treat PENDING as failure and do not retry** — you would double-spend. Show
  a pending state and observe the hash on chain.
- `FAILED` → `result.error` is `{ message, code }`; the documented code is
  `INSUFFICIENT_BALANCE`.
- `CANCELLED` → the user declined the confirmation prompt.

There is no idempotency key on these calls. `txHash` is your only deduplication
handle — record it before you retry anything.

## Testnet rule

`deposit` is **blocked** whenever the Mini App is connected to a testnet, because it
would debit real money from the Lemon app and credit test tokens. On a testnet, fund
the Mini App wallet from a faucet instead:

- Circle USDC faucet — https://faucet.circle.com/ (10 testnet USDC/hour)
- Alchemy faucets — https://www.alchemy.com/faucets (native ETH, POL, …)

Testnet chain ids are in `sandbox/lemon-cash-sandbox.yml`; the docs default is
`ChainId.POLYGON_AMOY` (80002).

## See also

- `sandbox/lemon-cash-sandbox.yml`
- `errors/lemon-cash-problem-types.yml`
- `data-model/lemon-cash-data-model.yml`
