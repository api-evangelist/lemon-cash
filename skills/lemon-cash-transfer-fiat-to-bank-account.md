---
name: Transfer fiat to a bank account from a Lemon Mini App
description: Initiate an ARS or PEN payout to a whitelisted bank destination with
  transferMoney, and track it by transferId.
api: '@lemoncash/mini-app-sdk'
api_docs: https://lemoncash.mintlify.app/functions/transfer-money
operations: [isLemonWebView, authenticate, transferMoney]
operation_kind: sdk-function
generated: '2026-08-04'
method: generated
source: https://lemoncash.mintlify.app/functions/transfer-money
---

# Transfer fiat to a bank account from a Lemon Mini App

`transferMoney` is the fiat leg of the Mini App SDK — added in v0.1.16 (2026-03-18).
It moves local currency to a bank destination, user-confirmed in the Lemon Cash app.

## Preconditions

- `authenticate` has succeeded this session.
- **The destination `paymentId` is whitelisted by Lemon.** This is not optional: an
  un-whitelisted destination fails. Adding or changing one requires contacting the
  Lemon team first.

## Call

```ts
import { transferMoney, Currency } from '@lemoncash/mini-app-sdk';

const result = await transferMoney({
  amount: '5000',
  currency: Currency.ARS,
  name: 'John Doe',
  paymentDestinationInformation: {
    paymentId: '0000003100010000000001',
  },
});
```

- `amount` — string.
- `currency` — `Currency.ARS` (Argentine peso) or `Currency.PEN` (Peruvian sol).
  Those are the only published values.
- `name` — optional recipient name.
- `paymentDestinationInformation.paymentId` — CBU/CVU for Argentina, CCI for Peru.
  The type allows additional attributes for other countries' banking systems.

## Handling the result

- `SUCCESS` → `result.data.transferId`.
- `PENDING` → `result.data.transferId` exists, settlement is not confirmed. Track by
  `transferId`; **do not re-issue the transfer**, there is no idempotency key.
- `FAILED` → `result.error` `{ message, code }` (`INSUFFICIENT_BALANCE` is documented).
- `CANCELLED` → the user declined the confirmation prompt.

Persist `transferId` at the moment you receive it, in both `SUCCESS` and `PENDING`.
It is the only handle you get on a money movement.

## See also

- `errors/lemon-cash-problem-types.yml`
- `conventions/lemon-cash-conventions.yml`
- `changelog/lemon-cash-changelog.yml` (transferMoney landed in 0.1.16)
