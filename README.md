# foundation-registry

Which contract addresses the Dwarves estate treats as live.

`registry.public.json` is a **generated projection**. Do not edit it by hand; edits are overwritten by the next publish.

## Why this is public

Everything here is public chain data: addresses, bytecode fingerprints, chain ids. All of it is readable from any RPC endpoint by anyone, and the swap frontend already ships the ICY addresses to every browser that loads it.

Publishing it means the deploy pipeline that consumes it needs no credential to read it, and no private copy that can silently drift.

The operational detail stays private. The full record lives in `dwarvesf/foundation-contracts` and carries which repos consume each address, why a given contract is flagged, and its deployment history. None of that is projected here.

## Fields

| Field | Meaning |
|---|---|
| `address` | The version identity. Contracts here are immutable, so an upgrade is a new address, never an edit |
| `bytecode_fingerprint` | First 32 hex of sha256 over the `eth_getCode` result. Detects that the code at an address changed |
| `status` | `live` or `superseded`. A consumer pinning a `superseded` address is a deploy-blocking error |
| `supersedes` | The address this one replaced. The upgrade chain, read backwards |
| `generated_at` | When this projection was produced. Consumers reject it past a staleness deadline |

## Verifying it yourself

Nothing here has to be trusted. Every fingerprint is reproducible from the chain:

```sh
curl -s -X POST https://mainnet.base.org \
  -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"eth_getCode","params":["<address>","latest"]}' \
  | jq -r .result | tr -d '\n' | shasum -a 256 | cut -c1-32
```

## What this is not

Not an endorsement that an address is safe to use, only a record of which one is current. A `live` contract can still carry known issues; that detail lives with the private record and with whoever operates it.
