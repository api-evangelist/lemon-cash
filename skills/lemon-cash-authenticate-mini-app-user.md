---
name: Authenticate a Lemon Mini App user
description: Sign a user into a Lemon Cash Mini App with SIWE, request the claims you
  need, and verify the signature server-side before trusting the wallet.
api: '@lemoncash/mini-app-sdk'
api_docs: https://lemoncash.mintlify.app/functions/authenticate
operations: [isLemonWebView, authenticate]
operation_kind: sdk-function
generated: '2026-08-04'
method: generated
source: https://lemoncash.mintlify.app/functions/authenticate
---

# Authenticate a Lemon Mini App user

Lemon Cash publishes no HTTP API. The only public contract is the Mini App SDK
(`@lemoncash/mini-app-sdk`), which talks to the Lemon Cash mobile app over a React
Native WebView bridge. Every operation named below is a real exported SDK function
documented at https://lemoncash.mintlify.app/.

## Preconditions

- `npm install @lemoncash/mini-app-sdk` (v0.1.16 at time of writing).
- A Mini App ID issued by the Lemon team — there is no self-serve dashboard yet.
- A backend that can mint and verify nonces.

## Steps

1. **Confirm you are inside Lemon.** `await isLemonWebView()`. It is async and returns
   `true` only when the native Lemon app confirms within a 1-second timeout. If it is
   `false`, render a fallback — every other function will error outside the WebView.
   (`isWebView` is the deprecated synchronous predecessor; do not use it.)

2. **Mint a nonce server-side.** At least 8 alphanumeric characters, unique per
   attempt. Store it with a timestamp, an expiry, and a `used` flag.

3. **Call `authenticate` immediately on entry**, before any other SDK function, and
   again on every entry and on every chain switch:

   ```ts
   const result = await authenticate({
     nonce,
     chainId: ChainId.POLYGON_AMOY,
     requirements: { claims: [ClaimKey.NAME, ClaimKey.EMAIL, ClaimKey.LEMONTAG] },
   });
   ```

   `chainId` is required. Request only the claims you actually need — the user sees
   your Terms & Conditions, Privacy Policy and the claim list before granting.

4. **Branch on `result`, do not catch.** The call resolves; it does not throw.
   - `TransactionResult.SUCCESS` → `result.data` has `wallet`, `grantedClaims`,
     `signature`, `message`.
   - `TransactionResult.FAILED` → `result.error` is `{ message, code }`; the documented
     code here is `INVALID_SIGNATURE`.
   - `TransactionResult.CANCELLED` → the user declined. This is not an error; return
     the user to a neutral state.

5. **Verify server-side before you trust anything.** Check the nonce exists, has not
   expired, has not been used, and matches the nonce inside the signed `message`. Then
   verify the SIWE signature (the docs use viem's `verifySiweMessage`, which honours
   ERC-6492 for contract wallets that are not deployed yet). Mark the nonce used.

## Claims available

`NAME`, `LAST_NAME`, `EMAIL`, `IS_PEP` (Argentina only), `LEMONTAG`,
`OPERATION_COUNTRY`. Returned as `MiniAppGrantedClaim` `{ key, value }` pairs.

## Rules that get a Mini App rejected

- Calling `deposit`, `withdraw`, `transferMoney` or `callSmartContract` before
  `authenticate`.
- Using anything other than `authenticate` as your login flow.
- Collecting user data outside the claims mechanism instead of via
  `requirements.claims`.

## See also

- `authentication/lemon-cash-authentication.yml`
- `errors/lemon-cash-problem-types.yml`
- `conventions/lemon-cash-conventions.yml`
