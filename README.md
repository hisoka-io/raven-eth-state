# eth-state

Private balance reads over Ethereum-style account state, built on the
[Raven](https://github.com/hisoka-io/raven) PIR framework through its public crates.

It stores a flat `address -> 32-byte big-endian balance` table and serves reads through InsPIRe,
which hides from the server which row of a shard is read. New writes land in a sidecar engine that
answers immediately and is folded into the main engine with an atomic swap. A client always queries
both engines and decrypts both answers, so the server does not learn which one held the value.
State is persisted as snapshots plus a write-ahead log and recovers after a crash.

## Status

A demo PIR adapter over Ethereum account state, not a production service. It is not published to crates.io.
The anonymity set is one shard (2048 accounts; the shard id is visible to the server), and the
client detects stale answers but does not verify them against a state root.

## Build and test

The Raven crates are path dependencies (`../../crates/...`), so this builds only inside a Raven
checkout, where it sits at `adapters/eth-state` as a git submodule:

```bash
git clone --recursive https://github.com/hisoka-io/raven.git
cd raven
cargo fmt --manifest-path adapters/eth-state/Cargo.toml -- --check
cargo clippy --manifest-path adapters/eth-state/Cargo.toml --all-targets -- -D warnings
cargo test --manifest-path adapters/eth-state/Cargo.toml --profile ci-test
```

Use the `ci-test` profile for tests: InsPIRe setup under the dev profile is slow enough to look
like a hang. A test that builds a full PIR instance peaks near 2 GiB, so the nextest profile
runs two at a time.

Run the demo, which serves and verifies private reads while writes are folded in:

```bash
cargo run --manifest-path adapters/eth-state/Cargo.toml --profile ci-test --bin demo
```

To drive a running anvil node instead of the synthetic corpus, build with the `anvil-e2e`
feature and pass `--mode anvil`; the node URL is read from `ANVIL_RPC_URL`, default
`http://127.0.0.1:8545`:

```bash
cargo run --manifest-path adapters/eth-state/Cargo.toml --profile ci-test \
  --features anvil-e2e --bin demo -- --mode anvil
```

Minimum Rust version: 1.91.

## License

[Apache-2.0](./LICENSE)
