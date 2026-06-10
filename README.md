# probopass

Centralized Protocol Buffers contract repository — the single source of truth
for gRPC contracts shared by all backend services.

This repository follows **contract-first development**: APIs are designed here
as protobuf contracts before any service implements them.

## What this repository is NOT

- No business logic
- No service implementations
- No database code

Only contracts (`.proto` files) and the Go code generated from them.

## Structure

```
proto/probopass/
├── common/v1/        # Shared building blocks (pagination, metadata, errors)
└── auth/v1/          # Authentication service contract
gen/go/               # Committed output of `buf generate`
```

Packages are versioned: `probopass.common.v1`, `probopass.auth.v1`.

## Prerequisites

- [Buf CLI](https://buf.build/docs/installation) (v1.x)
- Go 1.26+ (only to build/verify generated code)

## Commands

| Command | Purpose |
| --- | --- |
| `buf lint` | Lint protos against the STANDARD rule set |
| `buf format -w` | Format protos in place |
| `buf generate` | Regenerate `gen/go` (commit the result) |
| `buf breaking --against '.git#branch=main'` | Check for breaking changes vs main |
| `buf dep update` | Update the googleapis dependency lock |

CI enforces all of the above on every pull request, including that `gen/` is
up to date with the protos.

## Consuming the contracts (Go)

```sh
go get github.com/kurnhyalcantara/probopass
```

```go
import (
    authv1 "github.com/kurnhyalcantara/probopass/gen/go/probopass/auth/v1"
    commonv1 "github.com/kurnhyalcantara/probopass/gen/go/probopass/common/v1"
)
```

Each service package ships the gRPC client/server stubs (`*_grpc.pb.go`) and
grpc-gateway reverse-proxy handlers (`*.pb.gw.go`) for HTTP/JSON transcoding.

## Versioning policy

- Released package versions (`v1`) are immutable contracts: **no breaking
  changes**. Adding fields, messages, and RPCs is allowed.
- Breaking changes require a new package version (e.g. `probopass.auth.v2`
  in `proto/probopass/auth/v2/`), kept alongside `v1`.
- `buf breaking` runs in CI against `main` to enforce this.

## Adding a new domain

1. Create `proto/probopass/<domain>/v1/` and add `.proto` files with package
   `probopass.<domain>.v1`.
2. Add grpc-gateway `google.api.http` annotations to every RPC.
3. Run `buf lint && buf format -w && buf generate`.
4. Commit the protos together with the regenerated `gen/` output.
