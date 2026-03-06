# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Go package (`github.com/Causevest/hdkeychain`) implementing BIP-0032 hierarchical deterministic extended keys. It is a fork of `btcsuite/btcd/btcutil/hdkeychain` modified for the Causevest/XCV protocol.

## Build & Test Commands

```bash
# Run all tests
go test ./...

# Run a single test
go test -run TestName ./...

# Run benchmarks
go test -bench=. ./...

# Run with race detector
go test -race ./...
```

## Key Modifications from btcsuite Upstream

- **Hash function**: Uses BLAKE3 (truncated to 20 bytes) for `Hash160` instead of RIPEMD160(SHA256). The `Hash` method also uses BLAKE3 instead of double-SHA256.
- **Dependencies replaced**: Uses `github.com/Causevest/xcvec` (aliased as `btcec`) instead of `btcsuite/btcd/btcec`, and `github.com/Causevest/base58` instead of `btcsuite/btcutil/base58`.
- **chaincfg removed**: Network-specific functionality (`IsForNet`, `SetNet`, `CloneWithVersion`) is disabled/removed. Version bytes are hardcoded via `XCVersion()` returning `{0x00, 0x00, 0x00, 0x01}`.
- **Added methods**: `Address()` (returns raw 160-bit address bytes), `PubKey()` (returns compressed public key bytes), `Hash()` (returns blake3 hex fingerprint of serialized key).

## Architecture

Single-file implementation in `extendedkey.go`. The core type is `ExtendedKey` which represents both private and public HD keys (distinguished by the `isPrivate` field).

Key derivation flow: `GenerateSeed` → `NewMaster` → `Derive` (child keys) → `Neuter` (private→public).

Two derivation methods exist:
- `Derive()` — standard BIP-32 compliant
- `DeriveNonStandard()` — deprecated, preserves old buggy behavior (issue #172 from upstream)
