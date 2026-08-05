---
name: Call a smart contract from a Lemon Mini App
description: Execute single or batched contract calls from a Lemon Mini App, use
  Permit2 to skip the separate approve transaction, and write confirmation copy the
  user can actually read.
api: '@lemoncash/mini-app-sdk'
api_docs: https://lemoncash.mintlify.app/functions/call-smart-contract
operations: [isLemonWebView, authenticate, callSmartContract]
operation_kind: sdk-function
generated: '2026-08-04'
method: generated
source: https://lemoncash.mintlify.app/functions/call-smart-contract
---

# Call a smart contract from a Lemon Mini App

`callSmartContract` is the general-purpose on-chain escape hatch. The user confirms
every call inside the Lemon Cash app before it executes.

## Preconditions

- `authenticate` has succeeded this session; the wallet it returned is the signer.
- The target contract is deployed on the `chainId` you pass.

## Single call

```ts
await callSmartContract({
  contracts: [
    {
      contractAddress: '0x…',
      functionName: 'transfer',
      functionParams: ['0x…', '1000000'],
      value: '0',
      chainId: ChainId.POLYGON,
    },
  ],
});
```

`value` is the native currency to send with the transaction — pass `'0'` when none.
`functionParams` is an array; pass `[]` when the function takes no arguments.
`contractStandard` is optional (`ERC20` is the published value).

## Batched calls

Pass more than one entry in `contracts[]`. They are presented and executed together.

## Permit2 instead of a separate approve

Rather than making the user send an `approve` transaction first, attach a signed
permit. The SDK generates the EIP-712 signature, performs the one-time ERC-20 approval
to Permit2 if it has not happened yet, and executes with the permit.

```ts
permits: [{
  owner: wallet,            // the authenticated wallet
  token: '0x…',             // ERC-20 being approved
  spender: '0x…',           // MUST equal contractAddress
  amount: '1000000000000000000',  // smallest unit
  deadline: '1735689600',   // UNIX seconds
  nonce: '0',               // unique, replay guard
}]
```

`spender` must match `contractAddress` or the permit is meaningless. Keep `deadline`
short and `amount` bounded to what the call actually needs.

## Write the confirmation copy

The user's confirmation sheet is templated. Pass `titleValues` and
`descriptionValues` to fill `{{key}}` placeholders:

```ts
titleValues: { amount: '100', token: 'USDC' },
descriptionValues: { recipient: '0x…', network: 'Polygon' },
```

Any placeholder you do not supply is stripped from the final text — so "Swap
{{amount}} {{token}}" with only `amount` renders "Swap 100". Supply every key.

## Handling the result

- `SUCCESS` / `PENDING` → `result.data.txHash`. `PENDING` means unconfirmed, not
  failed; observe the hash rather than retrying.
- `FAILED` → `result.error` `{ message, code }` (`INSUFFICIENT_BALANCE` is documented).
- `CANCELLED` → the user declined.

## See also

- `conventions/lemon-cash-conventions.yml`
- `data-model/lemon-cash-data-model.yml`
- `conformance/lemon-cash-conformance.yml`
