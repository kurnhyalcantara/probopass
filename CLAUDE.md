# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

Centralized protobuf contract repository (contract-first development) — the single source of truth for gRPC contracts shared by all backend services. It contains **only** `.proto` contracts and the Go code generated from them. Never add business logic, service implementations, or database code here.

## Commands

```sh
buf lint                                      # lint (STANDARD rule set)
buf format -w                                 # format protos in place
buf generate                                  # regenerate gen/go (always commit the result)
buf breaking --against '.git#branch=main'     # breaking-change check vs main
buf dep update                                # refresh googleapis dep in buf.lock
go build ./...                                # verify generated code compiles
```

There are no tests; verification is `buf lint` + `buf build` + `go build ./...`. CI (`.github/workflows/ci.yaml`) fails on lint, format, breaking changes, or a stale `gen/` directory.

## Architecture

- `proto/probopass/<domain>/v<N>/` — contracts, one directory per domain per major version. Current domains: `common` (pagination, metadata, errors) and `auth` (AuthService).
- `gen/go/` — committed output of `buf generate` (go, grpc, and grpc-gateway plugins, all remote, `paths=source_relative`). Go module: `github.com/kurnhyalcantara/probopass`.
- Package naming mirrors the path: `probopass.<domain>.v<N>`.
- `go_package` is set by Buf managed mode (`buf.gen.yaml`); never hand-write `option go_package` in protos.
- `common/v1` is the only package other domains may import. Domains must not import each other.

## Contract conventions

- Every RPC takes/returns dedicated messages named `<Rpc>Request` / `<Rpc>Response` — never share or reuse request/response messages across RPCs.
- Every RPC must carry a grpc-gateway `option (google.api.http)` annotation (HTTP bindings come from the `buf.build/googleapis/googleapis` dep).
- Document every service, RPC, message, and field with comments — they are the API documentation consumers see.
- Breaking changes to a released `vN` package are forbidden. Evolve by adding fields/RPCs, or create `v(N+1)` alongside the old version. `buf breaking` enforces this in CI.
- After any proto change, run `buf format -w && buf lint && buf generate` and commit `gen/` in the same commit as the protos.
