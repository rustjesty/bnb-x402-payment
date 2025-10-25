# BNB x402 Payment

A simple, HTTP-native protocol for on-chain payments. x402 makes it trivial for web services to accept low-fee, fast blockchain payments with minimal integration.

"One line of server code to accept digital dollars — no fee, ~2s settlement, $0.001 minimum payment."

Quick example (Express):

```js
app.use(
  // How much you want to charge, and where you want the funds to land
  paymentMiddleware("0xYourAddress", { "/your-endpoint": "$0.01" })
);
```

See `examples/typescript/servers/express.ts` for a complete example.

## Table of contents

- TL;DR
- Getting started
- Design & philosophy
- Protocol overview
- Type specifications
- Facilitator HTTP API
- Schemes and networks
- Examples
- Testing
- Contributing
- License

## TL;DR

- Purpose: Provide a simple, permissionless, HTTP-native standard for paying web resources using blockchains.
- Integration: 1 line for servers, 1 function for clients.
- Guarantees: Trust-minimizing, chain & token agnostic, gasless for clients & resource servers (via facilitators).

## Getting started

Requirements
- Node.js v24 or higher (for the examples)

Install and run examples (from the repository root):

1. Install dependencies and build TypeScript examples

```powershell
# from repo root
cd examples/typescript
pnpm install; pnpm build
```

2. Run an example server

```powershell
cd examples/typescript/servers/express
# set PAY_TO or similar in .env to your address
pnpm dev
```

3. Run the matching client

```powershell
cd examples/typescript/clients/axios
# set PRIVATE_KEY in .env to the paying account
pnpm dev
```

You should see the client successfully request and receive the resource after paying.

## Design & philosophy

x402 was designed to solve shortcomings of existing online payment systems (high fees, friction, and poor programmatic control). Key principles:

- Open standard: no reliance on a single provider.
- HTTP-native: payments are carried alongside normal HTTP requests (no extra handshake outside normal client-server flows).
- Chain & token agnostic: extensible to new chains and signing schemes.
- Trust minimizing: facilitators do not gain unilateral control of funds.
- Easy to use: abstract blockchain complexity into facilitator services so clients and resource servers remain simple.

## Protocol overview

x402 reuses HTTP status 402 (Payment Required). Typical flow:

1. Client requests a resource.
2. Resource server responds 402 with a Payment Required object describing accepted payment options.
3. Client chooses one payment requirement and constructs a Payment Payload.
4. Client repeats the request including `X-PAYMENT: <base64(json)>`.
5. Resource server verifies the payload (locally or via a facilitator `/verify`).
6. If valid, server fulfills the request. Optionally the server settles the payment directly or asks a facilitator to `/settle`.
7. When settled successfully the resource server returns 200 and includes a `X-PAYMENT-RESPONSE` header (base64 JSON) describing settlement details.

This design allows resource servers to trade off immediate response vs stronger settlement guarantees.

## Type specifications

Payment Required Response (server -> client)

```json
{
  "x402Version": 1,
  "accepts": [ /* paymentRequirements */ ],
  "error": "" // optional
}
```

paymentRequirements

```jsonc
{
  "scheme": "string",           // logical payment scheme (eg. "exact")
  "network": "string",          // chain/network id
  "maxAmountRequired": "string",// maximum amount in atomic units
  "resource": "string",         // resource URL
  "description": "string",
  "mimeType": "string",
  "outputSchema": null | {},
  "payTo": "string",            // recipient address
  "maxTimeoutSeconds": 30,
  "asset": "string",            // e.g. ERC20 contract address where appropriate
  "extra": null | {}              // scheme-specific metadata
}
```

Payment Payload (sent by client in `X-PAYMENT`, base64-encoded JSON)

```json
{
  "x402Version": 1,
  "scheme": "exact",
  "network": "1",
  "payload": { /* scheme-specific */ }
}
```

## Facilitator HTTP API

A facilitator is an optional 3rd-party that performs verification and on-chain settlement so resource servers don't need node or wallet access.

POST /verify

Request body

```json
{
  "x402Version": 1,
  "paymentHeader": "<base64-payment-header>",
  "paymentRequirements": { /* object from server */ }
}
```

Response

```json
{
  "isValid": true,
  "invalidReason": null
}
```

POST /settle

Request body same as `/verify`.

Response

```json
{
  "success": true,
  "error": null,
  "txHash": "0x...",
  "networkId": "1"
}
```

GET /supported

Response

```json
{
  "kinds": [ { "scheme": "exact", "network": "1" } ]
}
```

## Schemes and networks

Schemes are logical payment methods (for example `exact`, which transfers a fixed amount). Each (scheme, network) pair defines how the payload is built, verified, and settled. Implementations and facilitators must declare supported pairs.

The repo contains a first-pass spec for `exact` on EVM chains at `specs/schemes/exact/scheme_exact_evm.md`.

## Examples

- `examples/typescript/servers/express.ts` — Express resource server using x402 middleware.
- `examples/typescript/clients/axios` — Example client that follows the 402 flow and pays via `X-PAYMENT` header.

Follow the Getting started section above to run them locally.

## Running tests

From the `examples/typescript` directory:

```powershell
cd examples/typescript
pnpm install
pnpm test
```

This runs the unit tests for the x402 packages.

## Contributing

Contributions are welcome. Please follow these guidelines:

- Read `CONTRIBUTING.md` (if present). If not present, open an issue to discuss large changes first.
- Keep scheme implementations modular — add new (scheme, network) handlers under `specs/schemes`.
- Add unit tests for new features and run `pnpm test`.

## Roadmap

See `ROADMAP.md` (if present) for longer-term plans and priorities.

## License

This project is provided under the terms of the LICENSE file in the repository root.

---

If you'd like, I can also:

- Add a compact architecture diagram (SVG) and badges (build, npm, license).
- Create a short FAQ section and troubleshooting tips for common integration issues.

Next: I'll verify the updated `README.md` content and report back.
