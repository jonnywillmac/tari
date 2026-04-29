# Repository Guidelines

## Project Overview

This is the Minotari/Tari Rust workspace. The default contribution branch is `development`; almost all new code should be based there unless a maintainer explicitly asks for `mainnet`, `nextnet`, or another release branch. The root `Cargo.toml` defines the workspace and pins the Rust edition and MSRV metadata.

Core areas:

- `applications/`: runnable Minotari node, wallet, miner, merge mining proxy, utilities, and MCP servers.
- `base_layer/`: blockchain, wallet, p2p, transaction, and consensus-facing crates.
- `comms/`: networking and DHT crates.
- `common/`, `common_sqlite/`, `infrastructure/`, `hashing/`: shared support crates.
- `clients/`: generated/client-facing gRPC clients.
- `integration_tests/`: cucumber-style integration tests.
- `docs/`: user, operator, release, and review documentation.

## Git Workflow

Start by checking branch state and remotes:

```bash
git status --short --branch
git remote -v
```

If the checkout is detached or behind upstream, fetch and switch to the current upstream development branch before making changes:

```bash
git fetch origin --prune
git switch development
git merge --ff-only origin/development
```

For external contributions, prefer a fork-based remote setup. Keep the canonical repository as `upstream` and use the contributor fork as `origin`:

```bash
git remote rename origin upstream
git remote add origin git@github.com:<your-user>/tari.git
git fetch upstream --prune
git switch development
git merge --ff-only upstream/development
git switch -c <short-topic-branch>
```

Do not push directly to `tari-project/tari` unless the user explicitly confirms they have maintainer access and intend to do that. Before pushing, verify that `origin` points at the fork:

```bash
git remote -v
git push -u origin <short-topic-branch>
```

Keep PRs focused. The contribution guide asks for small PRs, ideally under 400 lines of non-test code, with commit messages that explain why the change is needed.

## Upstream PR Checklist

Before opening or updating any PR to `tari-project/tari`, verify that the branch contains only the issue-specific work intended for upstream. Do not include fork-only notes, local agent configuration, unrelated cleanup, generated files, or opportunistic refactors.

Use these checks from the PR branch:

```bash
git fetch upstream --prune
git status --short --branch
git log --oneline upstream/development..HEAD
git diff --stat upstream/development...HEAD
git diff upstream/development...HEAD
```

Review the output and confirm:

- Every commit belongs to the issue being fixed.
- Every changed file is necessary for the issue, its tests, or directly relevant documentation.
- The PR stays focused on one job and is small enough to review comfortably.
- Commit messages and the PR description explain why the change is needed.
- Required tests, formatting, linting, and any focused manual verification are listed in the PR description.
- Breaking-change implications are explicitly called out when relevant: hard fork, data directory reset, network compatibility, transaction compatibility, wallet recovery, or public API changes.
- Fork-only files such as `.codex`, personal scripts, local notes, or private workflow guidance are not staged, committed, pushed, or included in the PR.

If unrelated work is present, split it before opening the PR. Prefer a fresh branch from `upstream/development` and cherry-pick only the intended commits:

```bash
git switch -c <clean-topic-branch> upstream/development
git cherry-pick <commit-sha>
```

## Toolchain And Dependencies

Use the repository toolchain files and CI configuration as source of truth:

- Root `rust-toolchain.toml` currently selects stable Rust.
- Root `Cargo.toml` contains `rust-version`; keep it aligned with CI if changed.
- Formatting in CI uses the pinned nightly from `.github/workflows/ci.yml`.
- Install `cargo-nextest` for the normal test path.
- Install `cargo-lints` before running the lint alias.

Useful setup commands:

```bash
cargo install cargo-nextest --locked --force
cargo install cargo-lints
rustup component add clippy
```

Only update Rust toolchain settings deliberately. If changing the toolchain, read `rust-toolchain.toml` first because it lists other files and build images that need coordinated updates.

## Build, Format, Lint, And Test

Prefer the aliases in `.cargo/config.toml`:

```bash
cargo ci-fmt
cargo ci-clippy
cargo ci-check
cargo ci-test-compile
cargo ci-test
```

To fix formatting:

```bash
cargo ci-fmt-fix
```

Common targeted commands:

```bash
cargo check -p <crate>
cargo test -p <crate> <test_name>
cargo nextest run -p <crate>
```

Full CI-style tests can be expensive. Run the narrowest relevant package tests first, then broader checks when the change affects shared behavior, consensus, networking, wallet storage, or public APIs.

Integration tests live under `integration_tests/` and can be run through:

```bash
cargo ci-cucumber
```

## Coding Standards

- Follow existing local patterns and crate boundaries.
- Match the style of the code immediately around the change. Before editing, read nearby modules, tests, error handling, logging, naming, async patterns, builder patterns, and helper APIs.
- Prefer existing project abstractions and crate-local helper functions over introducing new patterns.
- Keep edits surgical. Avoid broad formatting, naming churn, dependency swaps, or refactors unless they are necessary for the issue and justified in the commit or PR text.
- Keep production code panic-free where possible; avoid `unwrap` and `expect` outside tests or unreachable setup paths.
- Treat network, disk, config, CLI, dependency, and blockchain data as untrusted.
- Bound allocations and validate lengths before using data from untrusted sources.
- Use checked arithmetic where overflow or underflow could affect consensus, money, storage, or indexing.
- Avoid `usize` in serialized, hashed, consensus, or cross-platform data.
- For atomics, use `Ordering::SeqCst` unless there is a documented project-approved reason not to.
- Public methods and public functions should have useful doc comments when the purpose is not obvious.
- New source files need the BSD-3-Clause SPDX header described in `Contributing.md`.

## Consensus, Wallet, And Network Safety

Be conservative around:

- Consensus constants, block validation, transaction validation, hashing, serialization, and feature gates.
- Wallet migrations, output manager storage, transaction service storage, and recovery paths.
- P2P message parsing, peer banning behavior, DHT propagation, RPC boundaries, and timeout handling.
- Protobuf changes and generated client-facing API changes.

For these areas, add or update tests that cover malformed input, empty input, oversized input, duplicate input, and backwards compatibility where applicable. If a change can require a hard fork, data directory reset, network compatibility break, or wallet compatibility break, call it out in the PR description.

## Documentation And PR Notes

Primary docs to consult:

- `README.md` for build and runtime basics.
- `Contributing.md` for release branches, PR expectations, CI checks, labels, and license rules.
- `docs/src/reviewing_guide.md` for security review concerns.
- `.github/PULL_REQUEST_TEMPLATE.md` for required PR sections.

PR titles should follow Conventional Commits. Fill out testing notes with the exact commands run and describe how reviewers can verify the change locally.

Do not mention automated agents, coding assistants, or the tool that produced the change in commits, PR titles, PR descriptions, review comments, code comments, or documentation intended for upstream. Write all upstream-facing commentary as normal contributor communication focused on the code, motivation, risk, and verification.
