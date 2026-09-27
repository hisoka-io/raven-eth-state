# eth-state

Private balance reads over Ethereum-style account state. It stores a flat
`address -> 32-byte big-endian balance` table, serves reads through InsPIRe, which hides from
the server which row of a shard is read, and folds new writes in with a main plus sidecar engine
that swaps atomically.

This repository is part of the [Raven](https://github.com/hisoka-io/raven) PIR framework and is
its second adapter, next to Railgun: it uses the framework crates through their public API.

## Build

This repository depends on the Raven crates by path (`../../crates/...`), so it builds only
inside a Raven checkout, at `adapters/eth-state`, where Raven includes it as a git submodule:

```bash
git clone --recursive https://github.com/hisoka-io/raven.git
cd raven
cargo fmt --manifest-path adapters/eth-state/Cargo.toml -- --check
cargo clippy --manifest-path adapters/eth-state/Cargo.toml --all-targets -- -D warnings
cargo test --manifest-path adapters/eth-state/Cargo.toml --profile ci-test
```

Use the `ci-test` profile for tests: InsPIRe setup under the dev profile is slow enough to look
like a hang.

Run the demo, which serves and verifies private reads while writes are folded in:

```bash
cargo run --manifest-path adapters/eth-state/Cargo.toml --profile ci-test --bin demo
```

To drive a running anvil node instead of the synthetic corpus, build with the `anvil-e2e`
feature and pass `--mode anvil`; the node is read from `ANVIL_RPC_URL`, default
`http://127.0.0.1:8545`:

```bash
cargo run --manifest-path adapters/eth-state/Cargo.toml --profile ci-test \
  --features anvil-e2e --bin demo -- --mode anvil
```

Minimum Rust version: 1.91.

## License

[Apache-2.0](./LICENSE)
